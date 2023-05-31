# Tests

```ts
import React from "react";
import { act } from "react-dom/test-utils";
import { fireEvent, render, screen } from "../../tests";
import { FlashMessages, useFlashMessages } from "./FlashMessages";

test("Flash Messages", async () => {
  const TestComponent = () => {
    const { showFlashMessage } = useFlashMessages();
    return (
      <div>
        <button onClick={() => showFlashMessage("test text", "info")}>
          Show flash message
        </button>
      </div>
    );
  };

  await act(async () => {
    render(
      <div>
        <FlashMessages>
          <TestComponent />
        </FlashMessages>
      </div>
    );
  });

  const button = await screen.findByRole("button", {
    name: "Show flash message",
  });
  let flashMessage = screen.queryByText("test text");
  expect(flashMessage).not.toBeInTheDocument();

  await act(async () => {
    await fireEvent.click(button);
  });
  flashMessage = await screen.findByText("test text");
  expect(flashMessage).toBeInTheDocument();
});
```
