# Reusable Components 

**Reusable Components** let you turn a portion of an automation into a block that can be reused across scenarios. Instead of copying the same login, test-data setup, helper query, or shared validation nodes into several scenarios, you create a component once, define its inputs and output, publish it, and use it inside scenarios.

Think of a component as a **subflow with a contract**:

- the component receives input values;
- it runs its own sequence of nodes;
- it returns an output to the scenario that called it.

---

## When to Use

Use components when a sequence appears in many scenarios, for example:

- authenticating in a system;
- creating standard test data;
- querying data from an API or database and normalizing the result;
- running a shared business validation;
- encapsulating a small web automation used as setup for other tests.

Avoid components for actions that belong to only one scenario. If the logic appears once, keeping the nodes directly in the scenario is usually simpler.

---

## Where to Find

In the side menu, open **Components**.

The list lets you:

- create a new component;
- search by name;
- filter by category;
- filter by **Published** or **Draft** status;
- copy the component name or ID;
- delete components, respecting the user's permissions.

When a component is used by scenarios, QANode warns before deletion to avoid accidental removals.

---

## Creating a Component

1. Open **Components**.
2. Click **New Component**.
3. Enter:
   - **Name**: displayed in the list and in the scenario editor;
   - **Category**: group used to organize the component palette;
   - **Color**: visual color of the component on the canvas.
4. Create the component.

QANode opens the component editor. It uses the same visual base as the flow editor, but in **component mode**.

[Creating a Component](../../assets/images/criar-componente.mp4)

---

## Component Editor

Every component has two special nodes:

| Node | Purpose |
|------|---------|
| **Input** | Defines the data the scenario must send to the component |
| **Output** | Defines the value the component returns to the scenario |

These nodes are protected in the editor. They are part of the component contract and should not be removed.

Between **Input** and **Output**, add normal QANode nodes: Web, API, database, utilities, flow control, and any other nodes available in your environment.

---

## Component Inputs

In the **Input** node, configure the fields the component expects to receive.

| Field | Description |
|-------|-------------|
| **Field Name** | Name used to map the value in the scenario |
| **Type** | `string`, `number`, `boolean`, `object`, or `array` |
| **Required** | Defines whether the scenario must fill this value |
| **Test Data** | Value used when testing the component directly in the editor |

Example inputs:

| Name | Type | Required |
|------|------|----------|
| `email` | `string` | Yes |
| `password` | `string` | Yes |
| `role` | `string` | No |

[Setting component inputs](../../assets/images/entradas-componente.mp4)

During component execution, internal nodes can use these inputs through expressions, like any other node output:

```
{{ steps.Input.outputs.email }}
```

> Use simple, stable field names. This makes the component easier to consume and prevents hard-to-maintain expressions.

---

## Component Output

In the **Output** node, configure what the component returns to the scenario.

| Field | Description |
|-------|-------------|
| **Field Name** | Output name exposed to the scenario |
| **Type** | Expected value type |
| **Value** | Literal value or expression calculated from internal nodes |

[Setting component output](../../assets/images/saida-componente.mp4)

Example:

```
Name: token
Type: string
Value: {{ steps.login.outputs.json.token }}
```

The scenario that calls the component can use this value as the component node output.

> In this version, a component works with one main output in the Output node. If you need to return more than one value, return an object such as `{ token, userId, role }`.

---

## Testing a Component

Before publishing, test the component in its own editor.

1. Fill **Test Data** for required Input fields.
2. Click **Test Component**.
3. Follow the execution in the editor run panel.

Test executions validate the component in isolation. They do not replace official scenario runs that use the component.

If a required field has no test data, QANode blocks the component test until the value is provided.

[Testing a Component](../../assets/images/testar-component.mp4)

---

## Saving and Publishing

When saving a component, QANode stores:

- the internal component flow;
- the input contract;
- the output contract;
- name, category, and color.

For the component to appear in the scenario palette, it must be **Published**.

Important rules:

- draft components do not appear for use in scenarios;
- when the internal flow, inputs, or output change, the component requires review/publication again;
- publishing confirms that the current contract is ready to be used by other scenarios.

[Saving and Publishing a component](../../assets/images/salvar-e-publicar.mp4)

---

## Using in a Scenario

In the scenario editor:

1. Open the **Components** tab in the side palette.
2. Find the published component.
3. Drag the component to the canvas.
4. Connect it to other nodes.
5. Fill the input fields in the properties panel.

Inputs accept literals and expressions:

```
email: {{ variables.TEST_USER }}
password: {{ variables.TEST_PASSWORD }}
role: admin
```

During execution, QANode runs the component as part of the scenario. If a required input is empty, the scenario is blocked before running to prevent hard-to-diagnose failures.

[Using a component in a Scenario](../../assets/images/usar-em-cenario.mp4)

---

## Using Component Outputs

After the component runs, its output is available as an output of the component node in the scenario.

Example:

```
{{ steps.loginComponent.outputs.token }}
```

If the main output is an object:

```
{{ steps.prepareUser.outputs.result.userId }}
{{ steps.prepareUser.outputs.result.email }}
```

[Using Component Outputs](../../assets/images/usar-output-component.mp4)

---

## Permissions

In environments with access control, components use their own permissions:

| Permission | Allows |
|------------|--------|
| `component.view` | View the list and open components |
| `component.create` | Create and edit own components |
| `component.edit` | Edit components created by other users |
| `component.delete.own` | Delete own components |
| `component.delete` | Delete any component |
| `component.publish` | Publish components |

---

## Best Practices

- Use components for logic that is truly shared.
- Keep the contract small and clear.
- Name inputs and outputs with business terms, not internal technical details.
- Use categories to organize components by domain, such as `Login`, `Test data`, `Orders`, or `CRM`.
- Test the component in isolation before publishing.
- When changing a component used by many scenarios, validate at least one consuming scenario before releasing it to the team.
- Return an object when the output needs to carry several related values.

---

## Limitations and Care Points

- Published components can be used by many scenarios; contract changes may require updates in consuming scenarios.
- Draft components do not appear in the scenario editor palette.
- The Input and Output nodes are part of the component structure and should not be treated as regular nodes.
- If the component depends on web session, credentials, or variables, document that in the name, category, or in your team's usage convention.
