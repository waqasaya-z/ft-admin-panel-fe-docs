# Fees and Commissio Context
*** Micro-service |  Fg-Admin-Panel-Frontend | Fees and Commission Module *** 

## Component Name
**FeesAndCommissionsContext** — Context used in Fees and commission for managing module state primarily the website id.


### 1. Introduction: Purpose of the Context
This context is used for managing the fees and commissions data within your application. It serves as a centralized store that provides the necessary data and methods to access or modify details such as sites, countries, and site-specific information (siteDetails).

### 2. Context Interface: Defining the Structure
The context is defined by the interface IFeesAndCommissionsContext, which ensures that all necessary values are passed through the context provider. This interface also defines the structure for the context's state.

```typescript
interface IFeesAndCommissionsContext {
  siteDetails: any;
  setSiteDetails: (siteDetails: any) => void;
  loading: boolean;
  sites: any[];
  countries: any[];
}
```
- siteDetails: Holds website id of selected website from the main dropdown.
- setSiteDetails: A method to update the siteDetails state.
- loading: A boolean flag indicating whether data is being loaded.
- sites: An array of websites.
- countries: An array of countries.

### 3. Creating the Context
The context is created using createContext, which allows you to share data across components. Here, it is initialized with a default value of undefined.

```typescript
const FeesAndComissionsContext = createContext<IFeesAndCommissionsContext | undefined>(undefined);
```

- The default value being undefined ensures that the context is properly checked before usage, helping prevent errors if it is not properly wrapped by the provider.

### 4. The FeesAndCommissionsProvider Component
This component wraps the application with the context provider. It accepts children and value as props. The value prop provides the context values (like sites, countries) that are passed down from the parent layout.

```typescript
export const FeesAndComissionsProvider = ({ children, value }: { children: ReactNode; value: any }) => {
  const [siteDetails, setSiteDetails] = useState<any>(null);
  return (
    <FeesAndComissionsContext.Provider value={{ ...value, siteDetails, setSiteDetails }}>
      {children}
    </FeesAndComissionsContext.Provider>
  );
};
```

- children: The components that need access to the context.
- value: The data passed down (like sites and countries) from the parent component.

### 5. Usage of useContext to Access the Context

The custom hook useFeesAndCommissionContext uses the useContext hook to provide easy access to the context data. If the context is not properly provided, an error is thrown to help avoid runtime issues.

```typescript
export const useFeesAndCommissionContext = () => {
  const context = useContext(FeesAndComissionsContext);
  if (!context) {
    throw new Error('useFeesAndCommissionContext must be used within a FeesAndComissionsProvider');
  }
  return context;
};
```

### 6. Value Prop in FeesAndCommissionsProvider
The value prop passed into the provider component comes from the parent layout or higher-level component. This value typically includes the sites (websites) and countries data, which are shared across the application.

```typescript
<FeesAndCommissionsProvider value={{ sites, countries }}>
 {children}
</FeesAndCommissionsProvider>
```