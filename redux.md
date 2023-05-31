# Redux

Setup ---------------------------------------------------------------------------------

```ts
const middleware = [thunkMiddleware];

const migrations = {
  // Change plan progress logic
  0: (state: ReduxState & PersistState) => {
    const copy = { ...state };
    copy.plan.plans = [];
    copy.plan.lastFetchTimestamp = 0;
    copy.plan.lastPlanSelectionTimestamp = 0;
    return copy;
  },
};

const persistConfig = {
  key: "root",
  storage: AsyncStorage,
  version: 0,
  whitelist: [propertyOf<ReduxState>("auth"), propertyOf<ReduxState>("plan")],
  migrate: createMigrate(migrations as any, { debug: __DEV__ }),
};
const persistedReducer = persistReducer(persistConfig, mainReducer);

export const store = createStore(
  persistedReducer,
  composeWithDevTools(applyMiddleware(...middleware))
);
export const persistor = persistStore(store);
```

Reducer ---------------------------------------------------------------------------------

```ts
export default function mainReducer(
  state: ReduxState = initState,
  action: ReduxAction
): ReduxState {
  return {
    auth: authReducer(state.auth, action),
    system: systemReducer(state.system, action),
    surveys: surveysReducer(state.surveys, action),
  };
}

export default function systemReducer(
  state: SystemState = initState,
  action: ReduxAction
): SystemState {
  switch (action.type) {
    case SystemActionTypes.SET_INTERNET_CONNECTION_STATUS: {
      return {
        ...state,
        isOnline: action.payload,
      };
    }
    case SystemActionTypes.SET_DEVELOPER_MODE: {
      return {
        ...state,
        isDeveloperMode: action.payload,
      };
    }
    case SystemActionTypes.SET_CURRENT_DATE: {
      return {
        ...state,
        currentDate: {
          year: action.payload.year,
          month: action.payload.month,
          day: action.payload.day,
        },
      };
    }
    default:
      return state;
  }
}

export default function surveysReducer(
  state: SurveyReducerState = initState,
  action: ReduxAction
): SurveyReducerState {
  switch (action.type) {
    case SurveyActionTypes.SET_SURVEYS: {
      return { ...state, surveys: action.payload.surveys };
    }
    case SurveyActionTypes.SET_LAST_SURVEY_FETCH_TIMESTAMP: {
      return { ...state, lastFetchTimestamp: action.payload.timestamp };
    }
    case SurveyActionTypes.SET_SURVEY_STATE: {
      const { surveyId, surveyState, lastChangeTimestamp } = action.payload;
      const surveyToUpdate = state.surveys.find((s) => s.id === surveyId);
      if (surveyToUpdate) {
        const otherSurveys = state.surveys.filter((s) => s.id !== surveyId);
        const surveyWithUpdatedState = new Survey({
          ...surveyToUpdate,
          state: surveyState,
          lastChangeTimestamp: lastChangeTimestamp,
        });
        return { ...state, surveys: [...otherSurveys, surveyWithUpdatedState] };
      } else {
        return state;
      }
    }
    case SurveyActionTypes.UPDATE_SURVEY: {
      const updated = action.payload;
      const updatedSurveys = state.surveys.map((s) => {
        if (s.id !== updated.id) return s;
        else return updated;
      });
      return { ...state, surveys: updatedSurveys };
    }
    default:
      return state;
  }
}
```

Actions ---------------------------------------------------------------------------------

```tsx
/**
 * Replaces all surveys with a new set of surveys
 * @param surveys New surveys
 */
export const setSurveys = (surveys: ISurvey[]): SetSurveys => ({
  type: SurveyActionTypes.SET_SURVEYS,
  payload: {
    surveys,
  },
});

export const fetchSurveys = (refetchAll = false) => {
  return async (
    dispatch: Dispatch,
    getState: () => ReduxState
  ): Promise<void> => {
    const state = getState();
    if (state.auth.user) {
      const userId = state.auth.user.id;
      const lastFetchTimestamp = state.surveys.lastFetchTimestamp;
      try {
        const payload =
          await SurveyManager.getManager().fetchSurveysPayloadAsync(
            userId,
            refetchAll ? 0 : lastFetchTimestamp
          );
        const changedSurveys: Survey[] = [];
        const notChangedSurveyIds: string[] = [];
        payload.surveys.forEach((surveyPayload) => {
          try {
            if (surveyPayload.isChanged && surveyPayload.data) {
              changedSurveys.push(
                transformSurveyPayloadToLocalType(surveyPayload.data)
              );
            } else {
              notChangedSurveyIds.push(surveyPayload.surveyId);
            }
          } catch (e) {
            logError(
              ErrorCategory.Survey,
              "Error transforming survey " + surveyPayload.surveyId,
              {
                survey: surveyPayload,
              }
            );
          }
        });
        const notChangedOriginal = state.surveys.surveys.filter((s) =>
          notChangedSurveyIds.some((nchs) => nchs === s.id)
        );
        const newSurveys: Survey[] = [...changedSurveys, ...notChangedOriginal];

        if (newSurveys.length > 0) {
          dispatch(setSurveys(newSurveys));
        }

        dispatch(setLastSurveyFetchTimestamp(getCurrentTimestampInSecond()));
      } catch (e) {
        console.log(e);
        dispatch(
          showFlashMessage("Could not fetch surveys. " + e.toString(), "error")
        );
      }
    } else {
      logError(ErrorCategory.Survey, "fetchSurveys:: User not found");
    }
  };
};

export type ReduxAction =
    | AuthActions
    | SystemActions
    | SurveyActions

export enum SurveyActionTypes {
  SET_SURVEYS = "SET_SURVEYS",
  SET_LAST_SURVEY_FETCH_TIMESTAMP = "SET_LAST_SURVEY_FETCH_TIMESTAMP",
  SET_SURVEY_STATE = "SET_SURVEY_STATE",
  UPDATE_SURVEY = "UPDATE_SURVEY",
}

export type SurveyActions =
  | SetSurveys
  | SetLastSurveyFetchTimestamp
  | SetSurveyState
  | UpdateSurvey;

export interface SetSurveys {
  type: typeof SurveyActionTypes.SET_SURVEYS;
  payload: {
    surveys: ISurvey[];
  };
}
```

Selectors

```tsx
export const getSurveys = (state: ReduxState): ISurvey[] =>
  state.surveys.surveys;

export const getSurveyById = (
  state: ReduxState,
  id: string
): ISurvey | undefined => state.surveys.surveys.find((s) => s.id === id);

const surveys = useSelector(getSurveys)
```
