# Module 13 — Forms, React Hook Form, Zod, and Server Validation

> **Build goal:** implement registration/profile forms and admin product create/edit with accessible labels, client validation, server field errors, pending states, and safe preservation of user input. Use React Hook Form 7.89.0, resolvers 5.9.1, and Zod 4.6.5 from the verified stack.

## 1. A form has two boundaries

The browser form boundary improves feedback; the backend boundary protects business data. TypeScript describes source-level relationships, but neither TypeScript nor client validation prevents a crafted request. Validate again on the server.

A good form flow is:

```text
user input → normalize/parse → field feedback → submit
  → server validation/authorization → success or mapped field/domain errors
```

Use validation that teaches the user what to fix. Avoid vague “invalid input” messages and do not validate only on blur if it leaves the user unaware of required fields.

## 2. Zod schemas: input can differ from output

A form may collect strings while the API expects normalized domain values. Zod 4 can validate and transform at the boundary.

```ts
import { z } from "zod";

const productFormSchema = z.object({
  name: z.string().trim().min(2, "Enter at least two characters."),
  category: z.enum(["audio", "home", "office"]),
  price: z
    .string()
    .trim()
    .regex(/^\d{1,9}(\.\d{1,2})?$/, "Enter a price with up to two decimal places.")
    .transform((text) => {
      const [whole = "0", fraction = ""] = text.split(".");
      return Number(whole) * 100 + Number((fraction + "00").slice(0, 2));
    })
    .pipe(z.number().int().nonnegative()),
});

type ProductFormInput = z.input<typeof productFormSchema>;
type ProductFormOutput = z.output<typeof productFormSchema>;
```

`ProductFormInput.price` is a string for the human-entered USD amount; `ProductFormOutput.price` is an integer number of cents. The API model names that value `amountMinor`. The nine-digit limit keeps this example within a safe integer range, and parsing avoids binary floating-point money arithmetic.

## 3. React Hook Form integration

The resolver connects RHF submission to the runtime schema. The form owns dirty/touched/submission state; do not mirror those fields in Zustand or Query.

```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

function ProductForm({ initialValues }: { initialValues: ProductFormInput }) {
  const form = useForm<ProductFormInput, unknown, ProductFormOutput>({
    resolver: zodResolver(productFormSchema),
    defaultValues: initialValues,
    mode: "onBlur",
  });

  async function onSubmit(values: ProductFormOutput) {
    await saveProduct({
      name: values.name,
      category: values.category,
      price: { amountMinor: values.price, currency: "USD" },
    });
  }

  return (
    <form onSubmit={form.handleSubmit(onSubmit)} noValidate>
      <label htmlFor="product-name">Product name</label>
      <input id="product-name" {...form.register("name")} aria-invalid={Boolean(form.formState.errors.name)} />
      {form.formState.errors.name && (
        <p role="alert">{form.formState.errors.name.message}</p>
      )}
      {/* Category and price fields follow the same label/error relationship. */}
      <button type="submit" disabled={form.formState.isSubmitting}>
        {form.formState.isSubmitting ? "Saving…" : "Save product"}
      </button>
    </form>
  );
}
```

The example demonstrates the input/output generic relationship; the production component should give every field a unique `id`, use the shared TextField, connect `aria-describedby`, include help text, and announce submit-level outcomes. `noValidate` disables browser-native popups when the app renders its own validation feedback; decide deliberately rather than adding it automatically.

## 4. Server errors and safe user input

Map known server field errors into RHF's `setError` and keep a form-level error for unknown failures.

```tsx
try {
  await onSubmit(values);
} catch (error) {
  if (error instanceof ApiError && error.code === "VALIDATION_FAILED") {
    for (const field of error.fieldErrors) {
      const path = field.path[0];
      if (path === "name" || path === "category" || path === "price") {
        form.setError(path, { type: "server", message: field.message });
      }
    }
    return;
  }
  setRootError("We couldn't save the product. Try again.");
}
```

Allowlist field names; do not cast arbitrary server paths into form paths. Preserve safe text fields when validation fails. Clear passwords after auth failure if policy requires, and never log them. Server messages must be safe and not contain stack traces.

## 5. Controlled widgets and field arrays

Use RHF `register` for native inputs. Use `Controller` only when a third-party/custom widget is controlled and does not expose the native input contract. Use `useFieldArray` for dynamic repeated groups, with stable field IDs; never use an array index as the React key for a reorderable field row.

Async validation (for example checking a slug) should be debounced/cancelled, associate results with the exact value checked, and never let a stale response overwrite newer input. The server still enforces uniqueness on submit.

## 6. Accessible form behavior

- Every control has a programmatic label.
- Required/format instructions appear before submission and are not conveyed by color alone.
- Errors are connected with `aria-describedby`, set `aria-invalid`, and are announced appropriately.
- On failed submit, focus the first invalid field or an error summary with links to fields.
- Pending state prevents accidental duplicate submissions but does not trap keyboard users.
- Preserve entered safe values after server rejection.
- Use `autocomplete` tokens for account/profile fields; use `type="password"` and do not expose password values to telemetry.

## Debugging lab

- Form type says `ProductInput`, but malicious request sends `price: -100`: TypeScript is not running on the server; validate server-side.
- Price 12.34 becomes 12.339999…: avoid binary float as canonical money; parse/store minor units.
- Server returns `path: "role"` and form tries `setError(path as any)`: allowlist expected form fields.
- Error text is visible but screen reader doesn't announce: connect IDs and test focus/status behavior.
- RHF rerenders whole page on every keypress: subscribe only where needed with `formState`/field state; don't add memoization before checking actual render cost.

## Exercises

1. Build sign-in with email/password validation and generic credential errors.
2. Build profile edit with `displayName` and theme preference; preserve values on server 422.
3. Build product form with USD text input transformed to integer minor units.
4. Add a server-side uniqueness response to a slug field and protect against stale async results.
5. Add an accessible error summary and test keyboard focus after invalid submission.

## Summary and production tips

**Summary:** RHF owns form lifecycle; Zod parses input and infers input/output types; the backend validates independently. Server errors become field or form feedback.

```text
unknown form/API value → runtime schema → typed domain output → authorized server operation
```

- Avoid duplicating form values in global state.
- Normalize at the boundary and preserve raw text when the user needs to correct it.
- Keep validation messages specific, actionable, and connected to controls.
- Do not use client validation as a security boundary.
- Confirm the resolver version supports the selected Zod major and RHF generics.

## Official documentation

- [React Hook Form `useForm`](https://react-hook-form.com/docs/useform)
- [`register`](https://react-hook-form.com/docs/useform/register)
- [`formState`](https://react-hook-form.com/docs/useform/formstate)
- [`Controller`](https://react-hook-form.com/docs/usecontroller/controller)
- [`useFieldArray`](https://react-hook-form.com/docs/usefieldarray)
- [RHF resolvers — Zod](https://github.com/react-hook-form/resolvers#zod)
- [Zod](https://zod.dev/) and [Zod API](https://zod.dev/api)
- [WAI Forms Tutorial](https://www.w3.org/WAI/tutorials/forms/)
- [WCAG labels or instructions](https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html)
- [React `<form>`](https://react.dev/reference/react-dom/components/form)

## Readiness criteria

You can distinguish input from output types, parse decimal money without floats, integrate a resolver, map server errors through an allowlist, preserve safe input, and complete the form by keyboard and screen reader without relying on color alone.
