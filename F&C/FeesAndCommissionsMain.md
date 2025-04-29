# Fees and Commission Main Tabs
***Micro-service |  Fg-Admin-Panel-Frontend | Fees and Commission Module*** 

### Component Name
**FeesAndCommissionsMain.tsx** — Fees and Commissions main tabs that is directly loaded when we first visit the Fees and commissions module

## 1. Introduction: Purpose
The purpose of the FeesAndCommissionsMain component is to render a tab-based interface for managing and displaying the fees and commissions data. This interface includes multiple tabs such as "Service Fee," "Bank Transaction Fee," and others. Each tab loads a corresponding component for handling the data and logic associated with that particular fee type.

The TabContext, along with TabList, TabPanel, and Suspense, facilitates lazy loading of content to improve performance. The component utilizes the Material-UI library for UI components and applies state management to track the active tab. The ErrorBoundary ensures that any errors that occur during rendering are handled gracefully, preventing the entire component from crashing.

## 2. Importing Dependencies

```typescript
import TabContext from '@mui/lab/TabContext'
import TabList from '@mui/lab/TabList'
import TabPanel from '@mui/lab/TabPanel'
import { Card, CardContent, CircularProgress, Grid, Tab } from '@mui/material'
import React, { Suspense, SyntheticEvent, useState } from 'react'
import {
  FEES_AND_COMMISSIOS_MAIN_TABS,
  FEES_AND_COMMISSOINS_MAIN_TABS_COMPONENTS,
  FEES_COMMISSIONS_TABS
} from 'src/types/feescommissions.types'
import ErrorBoundary from 'src/utils/ErrorBoundary'
```

**Material-UI Components:**
- TabContext, TabList, TabPanel, and Tab from @mui/lab and @mui/material provide the necessary components for rendering and managing tabs.
- Card, CardContent, CircularProgress, and Grid are Material-UI components used for layout and styling, including a loading spinner (CircularProgress).

**React:**
- useState: React hook used for managing the state of the selected tab.
- Suspense: Enables lazy loading of components, allowing for improved performance by displaying a fallback UI (like a loading spinner) while the content is loading.
- SyntheticEvent: Used for handling events like tab changes.

**Custom Types:**
- FEES_AND_COMMISSIOS_MAIN_TABS: Defines all possible main tabs for the fees and commissions section.
- FEES_AND_COMMISSOINS_MAIN_TABS_COMPONENTS: A record that maps each tab to its corresponding component.
- FEES_COMMISSIONS_TABS: Specifies the tab values and labels.

**Error Handling:**
- ErrorBoundary: A component to handle JavaScript errors gracefully and provide a fallback UI.

## 3. Main Component: FeesAndCommissionsMain
This is the core component responsible for rendering the tabs and loading the content associated with each tab.

```typescript
const FeesAndCommissionsMain = () => {
  const [tabValue, setTabValue] = useState<FEES_AND_COMMISSIOS_MAIN_TABS>(FEES_COMMISSIONS_TABS.SERVICE_FEE.value)

  const handleTabChange = (event: SyntheticEvent, newValue: FEES_AND_COMMISSIOS_MAIN_TABS) => {
    setTabValue(newValue)
  }

  return (
    <Grid item xs={12}>
      <Card>
        <CardContent>
          <TabContext value={tabValue}>
            <TabList onChange={handleTabChange} variant='scrollable' scrollButtons='auto'>
              {Object.values(FEES_COMMISSIONS_TABS).map(tab => (
                <Tab key={tab.value} value={tab.value} label={tab.label.toUpperCase()} />
              ))}
            </TabList>
            <ErrorBoundary>
              <Suspense fallback={<CircularProgress />}>
                {Object.entries(FEES_AND_COMMISSOINS_MAIN_TABS_COMPONENTS).map(([key, Component]) => (
                  <TabPanel key={key} value={key}>
                    {key === tabValue && <Component />}
                  </TabPanel>
                ))}
              </Suspense>
            </ErrorBoundary>
          </TabContext>
        </CardContent>
      </Card>
    </Grid>
  )
}

export default FeesAndCommissionsMain
```

## 4. Key Concepts and Explanation
**Tab Context Setup**
- TabContext: This component is used to manage the current active tab value and provide that value to child components.
- TabList: Renders the list of available tabs. When a user selects a tab, the onChange event handler (handleTabChange) updates the tabValue state, which determines the active tab.

```typescript
<TabContext value={tabValue}>
  <TabList onChange={handleTabChange} variant='scrollable' scrollButtons='auto'>
    {Object.values(FEES_COMMISSIONS_TABS).map(tab => (
      <Tab key={tab.value} value={tab.value} label={tab.label.toUpperCase()} />
    ))}
  </TabList>
  ```

**Tabs**
- Dynamic Tabs: Object.values(FEES_COMMISSIONS_TABS) is used to dynamically generate the tabs from the predefined FEES_COMMISSIONS_TABS configuration. The Tab component is rendered for each tab type, and each tab’s value and label are mapped from this configuration.

```typescript
<Suspense fallback={<CircularProgress />}>
  {Object.entries(FEES_AND_COMMISSOINS_MAIN_TABS_COMPONENTS).map(([key, Component]) => (
    <TabPanel key={key} value={key}>
      {key === tabValue && <Component />}
    </TabPanel>
  ))}
</Suspense>
```

**Lazy Loading with Suspense**
- Suspense: React’s Suspense component is used here to enable lazy loading for the content of each tab. A fallback component (in this case, CircularProgress) is shown until the tab content has finished loading.
- Lazy Loading Components: For each tab, a component is dynamically loaded using the Object.entries(FEES_AND_COMMISSOINS_MAIN_TABS_COMPONENTS) method, which maps the components based on the selected tab. The content of the tab is only loaded when the tab is selected. This approach minimizes the initial load time of the page and ensures that only the active tab’s content is rendered.

**Conditional Rendering of Tab Content**
- TabPanel: A TabPanel component wraps the content of each tab. Only the content of the active tab (determined by tabValue) is rendered. The TabPanel ensures that only the component corresponding to the active tab is displayed.
- Optimized Rendering: {key === tabValue && <Component />} ensures that the component for the currently active tab is rendered, preventing unnecessary rendering of inactive tabs.

```typescript
<TabPanel key={key} value={key}>
  {key === tabValue && <Component />}
</TabPanel>
```


## 5. FEES_AND_COMMISSOINS_MAIN_TABS_COMPONENTS Record
This record maps each tab (defined by FEES_AND_COMMISSIOS_MAIN_TABS) to its corresponding component, which handles the specific logic and UI for that tab.

```typescript
export const FEES_AND_COMMISSOINS_MAIN_TABS_COMPONENTS: Record<
  FEES_AND_COMMISSIOS_MAIN_TABS,
  React.ComponentType<any>
> = {
  SERVICE_FEE: React.memo(ServiceFee),
  BANK_TRANSACTION_FEE: React.memo(BankTransactions),
  PAYPAL_TRANSACTION_FEES: React.memo(PaypalTransactionFee),
  VAR_PERCENTAGE_PER_COUNTRY: React.memo(VarPercentage)
}
```

- React.memo: Each component is wrapped in React.memo, which is a performance optimization that prevents unnecessary re-renders of components if their props have not changed. This is especially useful for components that might have complex render logic or are expensive to re-render.

## 6. FEES_AND_COMMISSIOS_MAIN_TABS Enum or Type
The FEES_AND_COMMISSIOS_MAIN_TABS enum or type defines all the possible tab values (such as SERVICE_FEE, BANK_TRANSACTION_FEE, etc.). These values are used throughout the component to determine which tab is active and which component to render.

## 7. Error Boundary for Robustness
The ErrorBoundary component is used to catch JavaScript errors in any of the child components. If an error occurs, it prevents the entire component tree from crashing and instead displays a fallback UI.

```typescript
<ErrorBoundary>
  <Suspense fallback={<CircularProgress />}>
    {Object.entries(FEES_AND_COMMISSOINS_MAIN_TABS_COMPONENTS).map(([key, Component]) => (
      <TabPanel key={key} value={key}>
        {key === tabValue && <Component />}
      </TabPanel>
    ))}
  </Suspense>
</ErrorBoundary>
```