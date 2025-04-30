# Affiliate Context
***Micro-service | Fg-Admin-Panel-Frontend | Affiliates Module***

## Component Name
**AffiliateContext.tsx** — Context used in the Affiliate module for managing affiliate state including selected affiliate ID, affiliate data, and a refetch trigger function.

## 1. Introduction: Purpose of the Context
This context is used for managing affiliate-related data within the application. It serves as a centralized state provider for affiliate ID, affiliate data, and a refetch function that can be triggered after an update or change. This ensures consistent access to these values across deeply nested components.

## 2. Context Interface: Defining the Structure
The context is defined by the interface `IAffiliateContext`, ensuring type safety and clarity on the data structure and functions made available through the context.

```typescript
interface IAffiliateContext {
  affiliate_id: number | undefined;
  setAffiliateId: Dispatch<SetStateAction<number | undefined>>;
  affliateData: AffiliatesT;
  setAffliateData: Dispatch<SetStateAction<AffiliatesT>>;
  refetchAffiliates: (() => Promise<void>) | null;
  setRefetchAffiliates: Dispatch<SetStateAction<(() => Promise<void>) | null>>;
}
```

- **affiliate_id:** Holds the selected affiliate's ID.
- **setAffiliateId:** A method to update the affiliate_id state.
- **affliateData:** Stores the details of the selected affiliate.
- **setAffliateData:** A method to update the affliateData object.
- **refetchAffiliates:** A function to manually trigger a data refresh for affiliates.
- **setRefetchAffiliates:** A setter to update the refetch function dynamically.

## 3. Creating the Context
- The context is created using createContext, initialized with undefined to enforce that it must be consumed within a  properly wrapped provider.

```typescript
const AffiliateContext = createContext<IAffiliateContext | undefined>(undefined);
```

## 4. The AffiliateProvider Component
This provider wraps its children and supplies all relevant state and methods related to the affiliate context.
```typescript
export const AffiliateProvider = ({ children }: { children: ReactNode }) => {
  const [affiliate_id, setAffiliateId] = useState<number | undefined>(undefined);
  const [affliateData, setAffliateData] = useState<AffiliatesT>({} as AffiliatesT);
  const [refetchAffiliates, setRefetchAffiliates] = useState<(() => Promise<void>) | null>(null);

  return (
    <AffiliateContext.Provider
      value={{
        affiliate_id,
        setAffiliateId,
        affliateData,
        setAffliateData,
        refetchAffiliates,
        setRefetchAffiliates
      }}
    >
      {children}
    </AffiliateContext.Provider>
  );
};
```

- **children:** The nested React components that require access to affiliate-related state.

## 5. Usage of useContext to Access the Context
The custom hook useAffiliateContext makes accessing the context convenient and ensures it is used correctly within the provider.

```typescript
export const useAffiliateContext = () => {
  const context = useContext(AffiliateContext);
  if (!context) {
    throw new Error('useAffiliateContext must be used within a AffiliateProvider');
  }
  return context;
};
```

This provides automatic error checking to prevent using the context outside of its provider.

## 6. Example Usage in Application

```tsx
import { useAffiliateContext } from 'src/context/AffiliateContext';

const SomeComponent = () => {
  const { affiliate_id, setAffiliateId, affliateData, refetchAffiliates } = useAffiliateContext();
  
  useEffect(() => {
    if (refetchAffiliates) {
      refetchAffiliates();
    }
  }, [refetchAffiliates]);
  
  return (
    <div>
      <p>Current Affiliate ID: {affiliate_id}</p>
      <button onClick={() => setAffiliateId(123)}>Set Affiliate ID</button>
    </div>
  );
};
```