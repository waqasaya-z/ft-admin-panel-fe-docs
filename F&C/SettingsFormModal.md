# Settings Form Modal
***Micro-service |  Fg-Admin-Panel-Frontend | Fees and Commission Module*** 

### Component Name
**SettingsFormModal.tsx** — A dynamic form modal component for adding or editing fees and commissions settings.

### Overall Description
`SettingsFormModal` is a flexible modal form used in the Fees and Commissions module ("Main" and "Settings" tabs).  
It dynamically loads fields, payload structures, validators, and endpoint URLs based on the type.  
It supports both **add** and **edit** functionalities and manages `website_id` association when needed.

---

### 1. Chunk: Import Statements
### Import Statements and Their Explanations

### React Imports
- **`React`** - The base React library required for JSX syntax and building React components.
- **`useEffect`** - A hook used for managing side effects in functional components (e.g., fetching data, DOM manipulation) after the component renders.

### Custom Components & Utilities
- **`GenericForm`** - A custom component used for rendering and handling form submissions with add/update functionality.
- **`SubmitHandler, useForm`** - Imported from `react-hook-form` for managing form state and validation. `useForm` handles form inputs, while `SubmitHandler` defines the form submission logic.
- **`yupResolver`** - Used to integrate `Yup` schema validation with `react-hook-form` to apply validation rules to the form.
- **`HttpStatusCode`** - Enum from `axios` used to handle standard HTTP status codes in API responses, for better error handling.
- **`toast`** - A utility from `react-hot-toast` for showing notification messages to users (success, error, etc.) in a toast format.
- **`FC_URL`** - A constant that holds the base URL for API requests related to the component, making it easier to manage and reuse across different files.
- **`useFeesAndCommissionContext`** - A custom hook that provides access to context data for managing fees and commissions, allowing the component to fetch and manipulate data within the context.
- **`formFields, payloadMapping`** - Utilities imported from the `payload` module that define the structure of the form fields and map the payload data, respectively, for API requests.
- **`FEES_COMMISSIONS_ALL_TYPES`** - TypeScript type definition for various fee and commission types, used to ensure proper type-checking when handling data.
- **`validationSchemaMapping`** - A mapping utility that associates each fee/commission type with a corresponding validation schema, ensuring proper validation is applied for each type.
- **`ResponsiveDialog`** - A custom component that renders a responsive dialog (modal), typically used for adding or updating fees and commissions in a consistent layout.


### Chunk 2: Component State Setup

```typescript
const { siteDetails } = useFeesAndCommissionContext()
const validationSchema = validationSchemaMapping[type as FEES_COMMISSIONS_ALL_TYPES]
const payload = payloadMapping[type as FEES_COMMISSIONS_ALL_TYPES]
const url = FC_URL[type]
const fields = formFields[type]
```

**Explanation:**
- **siteDetails:** Extracted from the useFeesAndCommissionContext() hook. This provides access to the website id.
- **validationSchema:** Retrieves the appropriate validation schema for the form based on the type value. The validationSchemaMapping object maps each fee/commission type to a specific validation schema, ensuring that the form is validated properly before submission.
- **payload:** This holds the data mapping for the current form, determined by the type. The payloadMapping object defines how the form data should be structured when preparing it for submission to the backend.
- **url:** Retrieves the API endpoint URL specific to the fee/commission type from FC_URL. This URL will be used for making API requests for adding or updating fees/commissions.
- **fields:** Defines the form fields for the selected type using the formFields object. This ensures that the correct set of fields is rendered based on the type, offering a dynamic form structure.

### Chunk 3: Form Setup with React Hook Form

```typescript
const {
  control,
  handleSubmit,
  formState: { errors },
  reset
} = useForm({
  resolver: yupResolver(validationSchema),
  defaultValues: payload
})

useEffect(() => {
  if (isEdit) {
    const editData = data.find(item => item.id === isEdit)
    if (editData) {
      const updatedPayload = { ...payload, ...editData }
      reset(updatedPayload)
    }
  } else {
    reset(payload)
  }
}, [isEdit, data, reset, payload])
```

**Explanation:**
- control: This is provided by React Hook Form and is used to connect your form fields to the hook's functionality. It enables easy control of input components such as text fields or checkboxes.
- handleSubmit: This function is used to handle form submission. It triggers validation and passes form data to the specified callback function when the form is valid.
formState: { errors }: React Hook Form provides the errors object to capture form validation errors. You can use this to display error messages next to form fields.
- reset: A method from React Hook Form that allows resetting the form fields to a specific set of values, typically used to populate the form with data (e.g., for editing an existing entry).
- useEffect Hook: This hook is used for side effects in the component. Specifically, it listens for changes to the isEdit flag, data, and payload, and updates the form values accordingly.
- If isEdit is true, it searches for an existing record in data and merges that record with the default payload. This ensures that if the user is editing an existing item, the form is pre-populated with the correct data.
- If isEdit is false, it resets the form with the default payload, allowing the user to fill in new data.

### Chunk 4: Form Submission

```typescript
const onSubmit: SubmitHandler<any> = async data => {
  let dataPayload

  if (isEdit) {
    dataPayload = { ...payload, ...data }
  } else {
    dataPayload = { ...payload, ...data, website_id: siteDetails }
  }

  if (
    (type === 'BANK_TRANSACTION_FEE' ||
      type === 'PAYPAL_ZONE_TRANSACTION_FEE' ||
      type === 'PAYPAL_COUNTRY_TRANSACTION_FEE') &&
    dataPayload.website_id
  ) {
    delete dataPayload.website_id
  }

  try {
    let response

    if (isEdit) {
      response = await updateFn(isEdit, dataPayload, url)
    } else {
      response = await createFn(dataPayload, url)
    }

    if (response?.status === HttpStatusCode.Created || response?.status === HttpStatusCode.Ok) {
      toast.success(!isEdit ? 'Data was recorded.' : 'Data was amended.')
      setOpenModal(false)
      if (fetchFn) fetchFn()
    } else {
      toast.error('Something went wrong')
    }
  } catch (error: any) {
    if (error.response && error.response.status === 409) {
      toast.error(!isEdit ? 'Duplicate entry detected.' : 'No changes were submitted')
    } else {
      toast.error('Something went wrong')
    }
  } finally {
    setTimeout(() => setIsEdit(0), 100)
    reset(payload)
  }
}
```

**Explaination:**
- onSubmit is triggered when the form is submitted.
- The function first constructs a dataPayload by merging the form data with the default payload. If it's an edit operation (isEdit is true), it updates the existing data; otherwise, it includes the website_id in the payload.
- For certain fee types (BANK_TRANSACTION_FEE, PAYPAL_ZONE_TRANSACTION_FEE, PAYPAL_COUNTRY_TRANSACTION_FEE), the website_id is removed from the payload if it exists.
- The API call is made using either updateFn (for edit) or createFn (for add), depending on whether the form is in edit mode.
- If the response status is Created or Ok, a success toast is shown, the modal is closed, and any data fetching (if required) is triggered via fetchFn.
- If there is an error (e.g., 409 status indicating a duplicate entry), an appropriate error message is shown.
- Finally, the isEdit state is reset, and the form is reset to its default values.


### Chunk 6: Form Handling Closure
```typescript
const onClose = () => {
  setTimeout(() => setIsEdit(0), 50)
  setOpenModal(false)
  reset(payload)
}
```

**Explanation:**
- The onClose function is triggered when the user closes the modal.
- It resets the isEdit state to 0, ensuring that any previous edit mode is cleared.
- The modal is closed by setting setOpenModal to false.
- Finally, the form is reset to the original payload values, ensuring no unsaved data remains in the form

### Chunk 7: Return Value and TSX Rendering

```typescript
return (
  <ResponsiveDialog open={openModal} onClose={onClose} title={title}>
    <GenericForm
      fields={fields}
      onSubmit={onSubmit}
      control={control}
      errors={errors}
      buttonLabel={isEdit ? 'Save' : 'Submit'}
      handleSubmit={handleSubmit}
      customStyle={{ marginBottom: '17px' }}
      skeletonSx={{ marginBottom: '17px' }}
    />
  </ResponsiveDialog>
)
```

### Explanation:

**Return Value:** This component renders a modal (`ResponsiveDialog`) that contains a form (`GenericForm`). The form allows users to submit or save fee and commission settings.
- The modal (`ResponsiveDialog`) has an `open` prop to control its visibility, and an `onClose` handler to close the modal.
- Inside the modal, the `GenericForm` is rendered with dynamic fields, form submission logic, validation, and error handling based on the form's state.
- The form also conditionally displays the button label as **Save** or **Submit** depending on whether the form is in "edit" mode (`isEdit`).


### Chunk 8: Error Handling

**Explanation:**

**Error Handling:**

- The form leverages the `errors` object provided by `react-hook-form` to manage validation errors. If there are any validation issues (such as missing or invalid data), they are captured in this `errors` object and can be used to display error messages next to the relevant fields.
- If an error occurs during the form submission (such as network issues or server errors), appropriate error messages are displayed to the user using `toast.error()` in the submission logic.
- The component also checks for error responses during API calls (e.g., duplicate entries or submission issues) and displays a corresponding error message through the toast system.



### Key Features

1. **Dynamic Form Handling**  
   - The form dynamically adjusts based on the fee and commission type, rendering the appropriate fields, validation schema, and payload structure.

2. **Edit and Create Modes**  
   - The component supports both adding new and editing existing fee and commission settings. It pre-populates the form with existing data when in "edit" mode and allows the user to submit new data when in "create" mode.

3. **Conditional Submit Button**  
   - The submit button label changes based on the mode: `Save` for editing existing data and `Submit` for adding new data.

4. **Form Validation with React Hook Form**  
   - The form uses `react-hook-form` for easy handling of form inputs, validation, and submission. It integrates `yup` validation to ensure data is correct before submission.

5. **Responsive Modal**  
   - The form is displayed inside a responsive dialog modal (`ResponsiveDialog`), making it suitable for various screen sizes and providing a consistent user experience.

6. **Error Handling and Notifications**  
   - The component provides user feedback via `toast` notifications for success or failure during the form submission process. Error messages guide the user if something goes wrong, such as duplicate entries or failed API calls.

7. **Contextual API Interaction**  
   - Based on whether the user is editing or adding new data, the component interacts with different API functions (`createFn` for new entries, `updateFn` for existing entries) and handles
