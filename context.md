# Context

Definitions

```tsx
import { AlertColor, Stack } from "@mui/material";
import {
  createContext,
  FC,
  useCallback,
  useContext,
  useMemo,
  useState,
} from "react";
import Alert from "@mui/material/Alert";
import { useFlashMessagesStyles } from "./FlashMessages.styles";
import dataCy from "@app/dataCy";
import { useTranslation } from "react-i18next";

export interface FlashMessage {
  id: number;
  message: string;
  type: AlertColor;
  timeoutId: number;
  dataTestId?: string;
}

export interface FlashMessagesProps {
  children?: React.ReactNode;
  maxVisibleMessages?: number;
}

export interface FlashMessageHandle extends FlashMessage {
  close: () => void;
}

export interface FlashMessagesContextProps {
  showFlashMessage: (
    message: string,
    type: AlertColor,
    dataTestId?: string
  ) => FlashMessageHandle;
  showErrorMessage: (error: unknown, dataTestId?: string) => FlashMessageHandle;
  handleClose: (id: number, timeoutId?: number) => void;
  flashMessages: FlashMessage[];
}

const DEFAULT_STATE: FlashMessagesContextProps = {
  showFlashMessage: () => null as unknown as FlashMessageHandle,
  showErrorMessage: () => null as unknown as FlashMessageHandle,
  handleClose: () => null,
  flashMessages: [],
};

export const FlashMessagesContext =
  createContext<FlashMessagesContextProps>(DEFAULT_STATE);
FlashMessagesContext.displayName = "FlashMessagesContext";

export const useFlashMessagesContextProvider =
  (): FlashMessagesContextProps => {
    const [flashMessages, setFlashMessages] = useState<FlashMessage[]>([]);

    const handleClose = useCallback((id: number, timeoutId?: number) => {
      if (timeoutId) {
        clearTimeout(timeoutId);
      }
      setFlashMessages((previousState) =>
        previousState.filter((fm) => fm.id !== id)
      );
    }, []);

    const showFlashMessage = useCallback(
      (
        message: string,
        type: AlertColor,
        dataTestId?: string
      ): FlashMessageHandle => {
        const id = Math.floor(Math.random() * 1000000);

        const timeoutId = setTimeout(() => {
          setFlashMessages((previousState) =>
            previousState.filter((fm) => fm.id !== id)
          );
        }, 5000);

        const msg: FlashMessage = {
          id,
          message,
          type,
          timeoutId,
          dataTestId,
        };

        setFlashMessages((previousState) => {
          return [...previousState, msg];
        });

        return {
          ...msg,
          close: () => handleClose(msg.id, msg.timeoutId),
        };
      },
      [handleClose]
    );

    const { t } = useTranslation();

    const showErrorMessage = useCallback(
      (error: unknown, dataTestId?: string) => {
        return showFlashMessage(
          (error as Error)?.message ?? t("errorMessage.general"),
          "error",
          dataTestId
        );
      },
      [showFlashMessage, t]
    );

    return useMemo(
      () => ({
        showFlashMessage,
        handleClose,
        flashMessages,
        showErrorMessage,
      }),
      [showFlashMessage, handleClose, flashMessages, showErrorMessage]
    );
  };

export function useFlashMessages() {
  return useContext(FlashMessagesContext);
}

export const FlashMessages: FC<FlashMessagesProps> = ({
  children,
  maxVisibleMessages = 5,
}) => {
  const ctx = useFlashMessagesContextProvider();
  const { classes } = useFlashMessagesStyles();

  return (
    <FlashMessagesContext.Provider value={ctx}>
      <>
        <Stack spacing={2} className={classes.stack}>
          {ctx.flashMessages.slice(-maxVisibleMessages).map((fm) => (
            <Alert
              key={fm.id}
              data-testid={
                fm.dataTestId
                  ? fm.dataTestId
                  : dataCy.notification.flashMessage(fm.type)
              }
              className={classes.message}
              onClose={() => ctx.handleClose(fm.id, fm.timeoutId)}
              severity={fm.type}
            >
              {fm.message}
            </Alert>
          ))}
        </Stack>
        {children}
      </>
    </FlashMessagesContext.Provider>
  );
};
```

Usage

```tsx
export const App: React.FC = () => {
  return (
    <ThemeProvider theme="default">
      <GlobalStyles />
      <React.Suspense fallback={<LoadingIndicator centered />}>
        <QueryClientProvider client={queryClient}>
          ...
          <FlashMessages>
            <React.Suspense fallback={<LoadingIndicator centered />}>
              <AppRoutes />
            </React.Suspense>
          </FlashMessages>
          ...
        </QueryClientProvider>
      </React.Suspense>
    </ThemeProvider>
  );
};

export interface LazyLoadableProps {
  isLoading: boolean;
  isError: boolean;
  error: Error | unknown | null;
  children?: React.ReactNode;
}

export const LazyLoadable: React.FC<LazyLoadableProps> = (props) => {
  const { isLoading, isError, error, children } = props;
  const { showErrorMessage } = useFlashMessages();

  React.useEffect(() => {
    if (!isLoading && isError && error) {
      const msg = showErrorMessage(error);
      return () => {
        msg.close();
      };
    }
  }, [isLoading, isError, error, showErrorMessage]);

  if (isLoading) {
    return <LoadingIndicator centered />;
  }

  return <>{children}</>;
};
```
