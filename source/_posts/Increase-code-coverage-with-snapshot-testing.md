title: Increase code coverage with snapshot testing
featured_image: files/images/featured/snapshot.jpg
summary: >-
  A blog post where I'm trying to convince you that snapshots are a great and
  easy way to increase code coverage.
tags: []
categories: []
date: '2022-04-16T23:19:07.000Z'
---
Snapshot testing is exactly what the name suggests. Take the component into the desired state and create a snapshot. Screenshots or a copy of the component’s HTML code are the most common types of snapshots. 
Each test has a ground truth snapshot. When you run the tests, the new snapshot is being compared to the ground truth. If there is a difference, the test will fail. Your job is to decide if the failure happened because of a bug or an expected behavior change. While the concept is easy to grasp, I have seen projects that fail to unlock their full potential. 
Some of the reasons include: 
- Spending very little time designing tests
- Test scenarios that don't increase code coverage
- Mocking too many things
- Not mocking the very thing that is crucial to the code under test

Each of these topics deserves a separate blog post. In this one, I decided to go over 3 examples that deal with problems that arise when using snapshots in unit tests. 
The first example comes from the Alert component. It shows a message given by the parent. When the type of alert is PageNotFound, it will show links to related pages. 
```tsx
type AlertProps = { type: "PageNotFound" | "Other"; message: string };

export default function Alert({ type, message }: AlertProps) {
  return (
    <>
      <div>{message}</div>
      {type === "PageNotFound" && <div>Related pages: </div>}
    </>
  );
}
```
Take a moment to think about how would you test this component?
You will notice that the `type` prop determines the shape of the output. 
I came up with this:
<table class="table table-hover">
<thead>
  <tr>
    <th class="border-0">Type</th>
    <th class="border-0">Message</th>
    <th class="border-0">Output</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>PageNotFound</td>
    <td>Something went wrong</td>
    <td>
    ```html
      <div>Something went wrong</div>
      <div>Related pages: </div>
    ```
    </td>
  </tr>
  <tr>
    <td>Other</td>
    <td>Something went wrong</td>
    <td>
        ```html
      <div>Something went wrong</div>
    ```
    </td>
  </tr>
</tbody>
</table>

and this is my implementation.
```ts
import React from "react";
import renderer from "react-test-renderer";
import Alert from "./Alert";

it("Renders PageNotFound alert", () => {
  const tree = renderer.create(<Alert type="PageNotFound" message="Something went wrong" />).toJSON();
  expect(tree).toMatchSnapshot();
});

it("Renders other alerts", () => {
  const tree = renderer.create(<Alert type="Other" message="Something went very very wrong" />).toJSON();
  expect(tree).toMatchSnapshot();
});
```
We have the desired code coverage for the alert component, but the component itself has a bug. The list of related pages is not displayed. 
Let's use a new component for that. This component will receive a page name, make an API request to get a list of related pages and display that list.
```tsx
import React, { useLayoutEffect, useState } from "react";

type RelatedPage = {
  name: string;
  url: string;
};

export default function RelatedPages({ pageName = "" }: { pageName?: string }) {
  const [relatedPages, setRelatedPages] = useState<RelatedPage[]>([]);

  const fetchRelatedPages = async () => {
    const response = await fetch(`https://example.com/related/${pageName}`);
    const json = await response.json();
    setRelatedPages(json);
  };

  useLayoutEffect(() => {
    fetchRelatedPages();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [pageName]);

  return (
    <div>
      <>
        <h2>Related Pages</h2>
        {relatedPages.map((page) => {
          return (
            <div key={page.name}>
              <a href={page.url}>{page.name}</a>
            </div>
          );
        })}
      </>
    </div>
  );
}
```
The happy scenario is straightforward, provide a page name, and check if the list from API gets shown.
```jsx
import React from "react";
import { rest } from "msw";
import { setupServer } from "msw/node";
import { render, waitFor, screen } from "@testing-library/react";
import RelatedPages from "./RelatedPages";

const mockedRelatedPages = [
  {
    name: "Page 1",
    URL: "https://www.google.com/",
  },
  {
    name: "Page 4",
    URL: "https://www.google.com/",
  },
  {
    name: "Page 99",
    URL: "https://www.google.com/",
  },
];

it("Shows the correct related pages", async () => {
  //Mock API for the  success scenario.
  const server = setupServer(
    rest.get("https://example.com/related/badpage", (req, res, ctx) => {
      return res(ctx.status(200), ctx.json(mockedRelatedPages));
    })
  );
  server.listen({
    onUnhandledRequest: "warn",
  });

  const { asFragment } = render(<RelatedPages pageName="badpage" />);
  await waitFor(() => {
    return expect(screen.getByText("Page 1")).toBeTruthy();
  });

  expect(asFragment()).toMatchSnapshot();
  server.close();
});
```
The first thing I did was to create a mock response to the related pages API call. Then with the help of [MSW](https://mswjs.io/) I started a server that listens to my request and returns the mock data. The test is waiting for "page 1" to appear before taking the snapshot. 
For the final example, we will create a bug report functionality. 
The component should display a button with the text: Report a problem. Click on the button should open a modal with a form comprising of name, email, and description of the problem. Users should have the option to either close the modal or submit the bug report. The modal should have text for successful and failed bug reports (API errors).
One naive implementation of the form might look like this:
```tsx
import React, { useState } from "react";

export default function BugReport() {
  const [openDialog, setOpenDialog] = useState(false);
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [description, setDescription] = useState("");
  const [error, setError] = useState<string | undefined>();
  const [success, setSuccess] = useState(false);

  async function handleSubmit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault();
    try {
      await fetch("https://example.com/bugreport", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          name: name,
          email: email,
          description: description,
        }),
      }).then((result) => {
        //Here body is not ready yet, throw promise
        if (!result.ok) throw result;
        return result.json();
      });
      setSuccess(true);
      setError(undefined);
    } catch (err) {
      setError("Something went wrong");
    }
  }

  return (
    <>
      <button onClick={() => setOpenDialog(true)}>Report a problem</button>
      <dialog open={openDialog}>
        <h4>Bug report!</h4>
        <br />
        {success && <p>Thank you for your report!</p>}
        {!success && (
          <>
            {error && <p>{error}</p>}
            <form onSubmit={handleSubmit}>
              <label>
                Name:
                <input
                  type="text"
                  name="name"
                  required
                  value={name}
                  onChange={(event) => setName(event.target.value)}
                />
              </label>
              <br />
              <label>
                Email:
                <input
                  type="email"
                  name="email"
                  required
                  value={email}
                  onChange={(event) => setEmail(event.target.value)}
                />
              </label>
              <br />
              <label>
                Description:
                <br />
                <textarea
                  name="description"
                  required
                  value={description}
                  onChange={(event) => setDescription(event.target.value)}
                />
              </label>
              <br />
              <input type="submit" value="Send report" />
            </form>
            <br />
          </>
        )}
        <button
          onClick={() => {
            setOpenDialog(false);
            setError(undefined);
            setSuccess(false);
          }}>
          Close
        </button>
      </dialog>
    </>
  );
}
```
Based on the problem description and the code, I identified four distinct states:
1. The "Report a problem" button
2. The initial bug report modal
3. Successful bug report
4. Failed bug report (ex: API errors)

![bug-report-states.svg](/files/images/posts/increase-code-coverage-with-snapshot-testing/bug-report-states.svg)
The first test will check if the Report a problem button gets shown to the user. 
The second one tests the open and close functionality. The third and fourth tests will verify the successful and failed bug report states.
```tsx
import React from "react";
import { rest } from "msw";
import { setupServer } from "msw/node";
import { render, waitFor, screen, fireEvent } from "@testing-library/react";
import BugReport from "./BugReport";

it("Shows the report a problem button", async () => {
  const { asFragment } = render(<BugReport />);
  expect(asFragment()).toMatchSnapshot();
});

it("Opens the report bug modal", async () => {
  const { asFragment } = render(<BugReport />);
  fireEvent.click(screen.getByText(/Report a problem/i));
  expect(asFragment()).toMatchSnapshot();
  fireEvent.click(screen.getByText(/Close/i));
  expect(asFragment()).toMatchSnapshot();
});

it("Submits a successful bug report", async () => {
  //Mock API for the  success scenario.
  const server = setupServer(
    rest.post("https://example.com/bugreport", (req, res, ctx) => {
      return res(ctx.status(200), ctx.json({ status: "success" }));
    })
  );
  server.listen({
    onUnhandledRequest: "warn",
  });
  const { asFragment } = render(<BugReport />);
  fireEvent.click(screen.getByText(/Report a problem/i));
  fireEvent.change(screen.getByLabelText("Name:"), { target: { value: "Tony Stark" } });
  fireEvent.change(screen.getByLabelText("Email:"), { target: { value: "tony@StarkIndustries.com" } });
  fireEvent.change(screen.getByLabelText("Description:"), {
    target: { value: "Help! I can't find the reset password page." },
  });
  fireEvent.click(screen.getByText(/Send report/i));
  await waitFor(() => {
    return expect(screen.getByText("Thank you for your report!")).toBeTruthy();
  });
  expect(asFragment()).toMatchSnapshot();
  server.close();
});

it("Submits an unsuccessful bug report", async () => {
  //Mock API for the failure scenario.
  const server = setupServer(
    rest.post("https://example.com/bugreport", (req, res, ctx) => {
      return res(ctx.status(400));
    })
  );
  server.listen({
    onUnhandledRequest: "warn",
  });
  const { asFragment } = render(<BugReport />);
  fireEvent.click(screen.getByText(/Report a problem/i));
  fireEvent.change(screen.getByLabelText("Name:"), { target: { value: "Tony Stark" } });
  fireEvent.change(screen.getByLabelText("Email:"), { target: { value: "tony@StarkIndustries.com" } });
  fireEvent.change(screen.getByLabelText("Description:"), {
    target: { value: "Help! I can't find the reset password page." },
  });
  fireEvent.click(screen.getByText(/Send report/i));
  await waitFor(() => {
    return expect(screen.getByText("Something went wrong")).toBeTruthy();
  });
  expect(asFragment()).toMatchSnapshot();
  server.close();
});
```
Snapshots are a great and easy way to increase code coverage. 
To get the most out of them, pay close attention to snapshot changes during code review. Follow the best coding practices to ensure you don't get lost in them when the project gets bigger. 
And don't be afraid to combine them with other types of tests! As great as they are, snapshots are not the solution to all your testing problems.