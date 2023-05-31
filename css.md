# CSS

We used tss-react https://www.npmjs.com/package/tss-react

Definition

```tsx
import { makeStyles } from "tss-react/mui";

export const useRolloutModalStyles = makeStyles({ name: "RolloutModal" })(
  (theme) => ({
    container: {
      display: "flex",
      flex: 1,
      minHeight: 550,
      minWidth: 650,
      flexDirection: "column",
      gap: theme.spacing(2),
    },
    horizontalContainer: {
      display: "flex",
      width: "100%",
      gap: theme.spacing(2),
    },
    controlsButton: {
      minWidth: 120,
    },
    expand: {
      display: "flex",
      flex: 1,
    },
    tableContainer: {
      border: "none",
      padding: 0,
      minWidth: 600,
      height: 430,
      marginBottom: theme.spacing(2),
    },
    tableHead: {
      boxShadow: "0px 2px 1px -1px rgba(0, 0, 0, 0.25)",
      backgroundColor: theme.palette.background.paper,
      zIndex: 1,
      position: "sticky",
      top: 0,
    },
    row: {
      "&:last-child td, &:last-child th": { border: 0 },
      "&:hover > th:first-of-type": {
        boxShadow: `inset 8px 0px ${theme.layout.table.row.color.accent}`,
      },
      "&:hover th, &:hover td": {
        backgroundColor: theme.layout.table.row.color.hover,
      },
    },
    cell: {
      padding: `${theme.spacing(1)} ${theme.spacing(2)}`,
      height: 44,
    },
    headerCell: {
      height: 30,
      paddingTop: 0,
      paddingBottom: 0,
    },
    header: {
      display: "flex",
      justifyContent: "space-between",
      alignItems: "center",
    },
    title: {
      flex: "0 0 0%",
      whiteSpace: "nowrap",
    },
    targetSummary: {
      display: "flex",
      padding: `${theme.spacing(2)} ${theme.spacing(0)}`,
      gap: theme.spacing(2),
    },
  })
);
```

Usage

```tsx
export const RolloutModal: FC<RolloutModalProps> = (props) => {
  const { classes } = useRolloutModalStyles()
  ...
  return (
    <Modal open={open} onClose={onClose}>
      <Box className={classes.header}>
        <Typography variant='h4' className={classes.title}>
          {title}
        </Typography>
        ...
  )
}
```
