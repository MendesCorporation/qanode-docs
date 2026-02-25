# Quick Start

In this guide, you will create your first automated test in QANode in just a few minutes. We will create a simple web test that navigates to a page, fills out a form, and verifies the result.

---

## Step 1: Create a Project

1. In the side menu, click **Projects**
2. Click the **+ New Project** button
3. Fill in:
   - **Name**: `My First Project`
   - **Status**: Active
4. Click **Create**

[Creating a project](../../assets/images/criar-projeto.mp4)

*Image: Project creation modal with name and status fields*

---

## Step 2: Create a Flow (Test Scenario)

1. Inside the project, click the **Scenarios** tab
2. Click **+ New Scenario**
3. Enter the name: `Successful Login`
4. You will be taken to the **Flow Editor**

[Empty flow editor](../../assets/images/editor-fluxo.mp4)
*Image: Flow editor with an empty canvas and the node palette on the left*

---

## Step 3: Add Nodes

### Adding a Web Flow Node

1. In the node palette on the left, locate the **Web** category
2. Drag the **Smart Locators** node onto the canvas
3. Click the node to open the properties panel on the right

### Configuring the Steps

We will use the practice site **the-internet.herokuapp.com**, a public page built for automation testing. The page itself displays the credentials: username `tomsmith` and password `SuperSecretPassword!`.

In the properties panel, configure the following steps:

#### Step 1: Navigate to the login page

1. Click **+ Add Step**
2. Select the **Navigate** action
3. In the **URL** field, enter: `https://the-internet.herokuapp.com/login`

#### Step 2: Fill in the username field

1. Click **+ Add Step**
2. Select the **Fill** action
3. Configure the locator:
   - **Method**: `getByLabel`
   - **Value**: `Username`
4. In the **Text** field, enter: `tomsmith`

#### Step 3: Fill in the password

1. Click **+ Add Step**
2. Select the **Fill** action
3. Configure the locator:
   - **Method**: `getByLabel`
   - **Value**: `Password`
4. In the **Text** field, enter: `SuperSecretPassword!`

#### Step 4: Click the login button

1. Click **+ Add Step**
2. Select the **Click** action
3. Configure the locator:
   - **Method**: `getByRole`
   - **Value**: `button`
   - **Name**: `Login`

#### Step 5: Verify the result

1. Click **+ Add Step**
2. Select the **Assert** action
3. Configure:
   - **Mode**: `textContains`
   - **Locator**: `getByText` → `You logged into a secure area!`
   - **Expected Text**: `You logged into a secure area!`

[Configured steps](../../assets/images/configurando-passos.mp4)
*Image: Smart Locators properties panel with 5 configured steps*

---

## Step 4: Save and Run

1. Click the **Save** button (disk icon) in the top bar
2. Click the **Run** button (play icon) ▶️

QANode will:
1. Open a browser (headless mode by default)
2. Navigate to `https://the-internet.herokuapp.com/login`
3. Fill in the Username field with `tomsmith`
4. Fill in the Password field with `SuperSecretPassword!`
5. Click the Login button
6. Verify that the message `You logged into a secure area!` appears on the page

### Execution Result

After execution, each step will show its status:

- ✅ **Green**: Step executed successfully
- ❌ **Red**: Step failed
- ⏭️ **Gray**: Step not executed (skipped)

[Execution result](../../assets/images/execucao.mp4)
*Image: Editor showing nodes with success/failure indicators and the results panel*

Click on any node to see details:
- **Logs**: Messages from each executed step
- **Outputs**: Extracted data (texts, values, etc.)
- **Screenshots**: Screen captures (if configured)
- **Duration**: Execution time for each step

---

## Step 5: Add Screenshots (Evidence)

To capture evidence during execution:

1. Expand any step in the properties panel
2. In the **Evidence** section, enable **Capture Screenshot**
3. Select the mode:
   - **Before**: Captures before executing the step
   - **After**: Captures after executing the step
   - **Both**: Captures before and after

Screenshots will be stored as execution artifacts and included in PDF reports.

---

## Next Steps

Congratulations! You have created and run your first automated test. Now explore:

- [Core Concepts](concepts.md) — Better understand the QANode structure
- [Flow Editor](../flow-editor/overview.md) — Master the visual editor
- [Node Reference](../nodes/flow-control/if.md) — Learn about all available nodes
- [Expressions](../expressions/expressions.md) — Use dynamic data in your tests
- [Test Suites](../suites/suites.md) — Group and schedule your tests

---

## API Example

Want to test an API? Here is a quick example using **JSONPlaceholder**, a free public REST API for testing:

1. Drag an **HTTP Request** node onto the canvas
2. Configure:
   - **Method**: `GET`
   - **URL**: `https://jsonplaceholder.typicode.com/users/1`
3. Drag an **If** node and connect it to the HTTP Request output
4. Configure the condition: `{{ steps["http-request"].outputs.status === 200 }}`
5. On the **true** output, add a **Log** node with the message: `User found: {{ steps["http-request"].outputs.json.name }}`

This flow makes a GET request to JSONPlaceholder and checks if the status is 200. If so, it displays the returned user's name (`Leanne Graham`).
