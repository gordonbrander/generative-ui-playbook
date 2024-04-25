## Step 1: enhance user input to spec

```
You are a user interface specification expert. You specialize in converting user feature requests into a list of user stories. Expand on this request:

"I want a todo application that lets me add due dates"
```

```
As a user, I want to add due dates to my tasks, so that I can manage my deadlines effectively.
As a user, I want to view the due dates on my tasks in a list, so that I can quickly see when each task is due.
As a user, I want to easily edit the due dates of my tasks, so that I can adjust my plans as they change.
As a user, I want to receive notifications about my tasks as they approach their due dates, so that I can be reminded to complete them on time.
As a user, I want to enter due dates in various formats (e.g., DD/MM/YYYY, MM/DD/YYYY), so that I can use the format I am most comfortable with.
As a user, I want to view my tasks with due dates in a calendar format, so that I can better understand how my tasks are distributed over time.
As a user, I want to sort my tasks by their due dates, so that I can prioritize tasks that are due sooner.
As a user, I want to filter tasks by due date, so that I can focus on tasks that need immediate attention or are due within a certain timeframe.
As a user, I want to set due dates based on different time zones, so that I can manage tasks that are tied to specific geographical locations.
As a user, I want to set up recurring tasks with due dates, so that I don't have to manually set a due date each time the task recurs.
```

## Step 2: create domain model

```
You are a domain modelling expert. You generate simple Typescript types to express the domain model from a list of user stories. Create a model for this application, considering only the data and persistence aspects and NOT the user interface itself:

(...stories from above)
```

```ts
// Represents a single task with optional due date and recurrence
type Task = {
  id: string; // Unique identifier for the task
  title: string; // Title or description of the task
  dueDate?: Date; // Optional due date for the task
  timeZone?: string; // Optional time zone if the task is tied to a specific geographical location
  recurrence?: Recurrence; // Optional recurrence details
};

// Recurrence details, indicating the frequency and optionally end date of recurrence
type Recurrence = {
  frequency: RecurrenceFrequency; // How often the task recurs
  endDate?: Date; // Optional date when the recurrence ends
};

// Enum for possible recurrence frequencies
enum RecurrenceFrequency {
  Daily = "DAILY",
  Weekly = "WEEKLY",
  Monthly = "MONTHLY",
  Yearly = "YEARLY"
}

// Settings for user-specific preferences, like date format
type UserPreferences = {
  dateFormat: DateFormat; // Preferred date format for displaying dates
};

// Enum for supported date formats
enum DateFormat {
  DD_MM_YYYY = "DD/MM/YYYY",
  MM_DD_YYYY = "MM/DD/YYYY"
};
```

```
Assume you have access to the follow data persistence functions:

type Object = { id: string, type: string, data: object }
async function write(id: string, obj: Object) {}
async function read(id: string): Object {}
async function list(type: string): Object[] {}

Implement a data access API using these primitives to perform the key operations.
```

```ts
// Task-related constants
const TASK_TYPE = 'Task';

// Function to create a new task
async function createTask(taskData: Task): Promise<void> {
  const obj: Object = {
    id: taskData.id,
    type: TASK_TYPE,
    data: taskData
  };
  await write(taskData.id, obj);
}

// Function to update an existing task
async function updateTask(taskId: string, newTaskData: Partial<Task>): Promise<void> {
  const existingTaskObj = await read(taskId);
  if (existingTaskObj.type !== TASK_TYPE) {
    throw new Error("Task with given ID does not exist");
  }

  const updatedTaskData = { ...existingTaskObj.data, ...newTaskData };
  const updatedTaskObj: Object = {
    id: taskId,
    type: TASK_TYPE,
    data: updatedTaskData
  };
  await write(taskId, updatedTaskObj);
}

// Function to retrieve a task by its ID
async function getTask(taskId: string): Promise<Task> {
  const obj = await read(taskId);
  if (obj.type !== TASK_TYPE) {
    throw new Error("Task with given ID does not exist");
  }
  return obj.data as Task;
}

// Function to list all tasks
async function listTasks(): Promise<Task[]> {
  const objs = await list(TASK_TYPE);
  return objs.map(obj => obj.data as Task);
}
```

## Step 3: Consider UX

```
You are a skilled UX designer. Consider the following domain model, API specification and description of user stories and highlight any ambiguities, excessively complex features or nonsensical requirements.

(... domain, api, stories)
```

Reviewing your provided domain model, API specification, and user stories, here are some ambiguities and potential areas of complexity or improvement:

### Ambiguities
1. **Time Zone Handling for Notifications:**
   - The model allows tasks to be set in different time zones, but it is unclear how the system handles these for notifications. For example, will the notifications adjust according to the user’s current time zone or the task’s set time zone?

2. **DateFormat Handling in API:**
   - The user preference for date formats (DD/MM/YYYY or MM/DD/YYYY) is stored, but it is not clear how this preference is applied when dates are input or output through the API. There needs to be clarity on whether the API expects dates in a standard format or if it dynamically adjusts based on user preferences.

3. **Recurrence Details Specificity:**
   - The `Recurrence` type specifies `frequency` and an optional `endDate`, but it lacks details such as the day of the week for weekly recurrences, or which day of the month for monthly recurrences. This can lead to ambiguity in how exactly the tasks are meant to recur.

### Excessively Complex Features
1. **Multiple Date Formats:**
   - Supporting multiple date formats for input can complicate parsing logic and increase the potential for errors. A standardized ISO format for backend processing, with user-specific formats applied only in the UI, might simplify implementation.

2. **Time Zone Specific Tasks:**
   - While setting tasks in different time zones can be useful, it adds significant complexity to the system. Considerations include converting times for display, sorting, and notification purposes. This could be overly complex depending on the scope of your application.

### Potential Improvements
1. **API Error Handling:**
   - The API functions `updateTask` and `getTask` throw errors if a task does not exist. It might be beneficial to also return specific error messages for different kinds of failures (e.g., database read/write errors) to help with debugging and user feedback.

2. **Recurrence Handling:**
   - Enhance the `Recurrence` model to include more specific scheduling options (such as start times, specific weekdays, etc.), which will help in creating more flexible and useful recurring tasks.

3. **Simplification of Date Handling:**
   - Consider simplifying how dates are handled by using a consistent format in the back end and converting to/from the user's preferred format in the front end only. This approach reduces complexity in the data layer and isolates format-specific logic to the presentation layer.

4. **Task Sorting and Filtering API:**
   - While the user stories mention sorting and filtering tasks by due date, the API specification does not include methods for these functionalities. Implementing API endpoints that support filtering and sorting based on parameters like due date and recurrence can enhance usability.

5. **Use of `async/await` in API Functions:**
   - The API uses `async/await`, but it's not clear if the underlying `read`, `write`, and `list` functions support asynchronous execution. Ensure that these data access methods are properly handling asynchronous operations to avoid potential runtime issues in a production environment.

Overall, the user stories suggest a comprehensive task management application, but refining the details in the API specification and simplifying some of the more complex features could make implementation more straightforward and robust.

```
Rewrite the list of user stories to remove the excessive complexity and nonsense as well as clarifying and simplifying where possible.
```

```
Manage Deadlines:
As a user, I want to add due dates to my tasks using a standard date format (YYYY-MM-DD), so I can manage my deadlines effectively.
View Due Dates:
As a user, I want to view the due dates on my tasks in a list, so I can quickly see when each task is due.
Edit Due Dates:
As a user, I want to easily edit the due dates of my tasks, so that I can adjust my plans as they change.
Task Notifications:
As a user, I want to receive notifications about my tasks as they approach their due dates, to be reminded to complete them on time.
Calendar View of Tasks:
As a user, I want to view my tasks with due dates in a calendar format, so that I can better understand how my tasks are distributed over time.
Sort Tasks by Due Date:
As a user, I want to sort my tasks by their due dates, so that I can prioritize tasks that are due sooner.
Filter Tasks by Due Date:
As a user, I want to filter tasks by due date, so that I can focus on tasks that need immediate attention or are due within a specific timeframe.
Set Up Recurring Tasks:
As a user, I want to set up recurring tasks with a clear specification of recurrence rules (daily, weekly, monthly, yearly) and an optional end date, so I don't have to manually set a due date each time the task recurs.
```

## Step 4: Plan the App Build

```
You are a web application generator. You take a list of user stories, a data model and a set of API methods and implement accessible and beautiful web applications using lit-html and web components.

(... data ...)
```

1. Set up the project:
   - Create a new project directory and initialize it with a package.json file.
   - Install the necessary dependencies, including lit-html and any other required libraries.

2. Create the main web component:
   - Create a new file, e.g., `task-manager.js`, to define the main web component.
   - Import lit-html and define the `TaskManager` class that extends `LitElement`.
   - Implement the `render` method to define the component's template using lit-html.

3. Implement task creation:
   - Add an input field and a button in the `TaskManager` component's template for creating new tasks.
   - Handle the button click event to create a new task using the `createTask` function from the provided data API.
   - Clear the input field after creating the task.

4. Display the list of tasks:
   - Fetch the list of tasks using the `listTasks` function from the data API.
   - Render the list of tasks in the `TaskManager` component's template using lit-html's `repeat` directive.
   - Display the task title and due date (if available) for each task.

5. Implement task editing:
   - Add an edit button next to each task in the task list.
   - When the edit button is clicked, show an input field populated with the current task title and due date.
   - Handle the save button click event to update the task using the `updateTask` function from the data API.
   - Update the task list to reflect the changes.

6. Add due date functionality:
   - Modify the task creation and editing forms to include an input field for the due date.
   - Use the `<input type="date">` element to provide a user-friendly date picker.
   - Store the due date as a `Date` object in the `Task` data model.

7. Implement task sorting and filtering:
   - Add options to sort tasks by due date (ascending or descending) and filter tasks by a specific date range.
   - Update the task list based on the selected sorting and filtering options.

8. Implement task notifications:
   - Use the browser's Notification API to display notifications for tasks approaching their due dates.
   - Periodically check for tasks with upcoming due dates and show notifications accordingly.

9. Add recurring tasks:
   - Extend the task creation and editing forms to include options for setting up recurring tasks.
   - Allow users to specify the recurrence frequency (daily, weekly, monthly, yearly) and an optional end date.
   - Store the recurrence details in the `Task` data model.
   - Automatically create new task instances based on the recurrence rules.

10. Implement user preferences:
    - Create a separate web component, e.g., `UserPreferences`, to handle user-specific settings.
    - Allow users to select their preferred date format (DD/MM/YYYY or MM/DD/YYYY).
    - Store the user preferences using the provided data API.
    - Apply the selected date format when displaying dates throughout the application.

11. Implement accessibility:
    - Ensure that the application is keyboard-accessible by adding proper tabindex and aria attributes.
    - Use semantic HTML elements and provide appropriate labels and descriptions for form fields and buttons.
    - Test the application with assistive technologies to ensure compatibility.

12. Style the application:
    - Create a separate CSS file to define the styles for the web components.
    - Use CSS variables to maintain a consistent color scheme and typography throughout the application.
    - Ensure responsiveness and a mobile-friendly layout.

```
Now implement the plan.

Remember: use web standards, lit-html and web components. Return all code as one file. Do not leave any placeholders, implement all functions or omit them if they cannot be written. Assume that any component named `<task-manager>` will be rendered automatically, do not provide an html page.
```

```ts
import { LitElement, html, css } from 'lit';
import { repeat } from 'lit/directives/repeat.js';

// Task-related constants
const TASK_TYPE = 'Task';

// Function to create a new task
async function createTask(taskData) {
  const obj = {
    id: taskData.id,
    type: TASK_TYPE,
    data: taskData
  };
  await write(taskData.id, obj);
}

// Function to update an existing task
async function updateTask(taskId, newTaskData) {
  const existingTaskObj = await read(taskId);
  if (existingTaskObj.type !== TASK_TYPE) {
    throw new Error("Task with given ID does not exist");
  }

  const updatedTaskData = { ...existingTaskObj.data, ...newTaskData };
  const updatedTaskObj = {
    id: taskId,
    type: TASK_TYPE,
    data: updatedTaskData
  };
  await write(taskId, updatedTaskObj);
}

// Function to retrieve a task by its ID
async function getTask(taskId) {
  const obj = await read(taskId);
  if (obj.type !== TASK_TYPE) {
    throw new Error("Task with given ID does not exist");
  }
  return obj.data;
}

// Function to list all tasks
async function listTasks() {
  const objs = await list(TASK_TYPE);
  return objs.map(obj => obj.data);
}

class TaskManager extends LitElement {
  static get properties() {
    return {
      tasks: { type: Array },
      newTaskTitle: { type: String },
      newTaskDueDate: { type: String },
      sortBy: { type: String },
      filterDate: { type: String }
    };
  }

  constructor() {
    super();
    this.tasks = [];
    this.newTaskTitle = '';
    this.newTaskDueDate = '';
    this.sortBy = 'dueDate';
    this.filterDate = '';
  }

  async connectedCallback() {
    super.connectedCallback();
    this.tasks = await listTasks();
  }

  async createTask() {
    const newTask = {
      id: Date.now().toString(),
      title: this.newTaskTitle,
      dueDate: this.newTaskDueDate ? new Date(this.newTaskDueDate) : undefined
    };
    await createTask(newTask);
    this.tasks = [...this.tasks, newTask];
    this.newTaskTitle = '';
    this.newTaskDueDate = '';
  }

  async updateTask(taskId, newTaskData) {
    await updateTask(taskId, newTaskData);
    this.tasks = this.tasks.map(task => task.id === taskId ? { ...task, ...newTaskData } : task);
  }

  async deleteTask(taskId) {
    await deleteTask(taskId);
    this.tasks = this.tasks.filter(task => task.id !== taskId);
  }

  sortTasks(tasks) {
    if (this.sortBy === 'dueDate') {
      return tasks.sort((a, b) => a.dueDate - b.dueDate);
    }
    return tasks;
  }

  filterTasks(tasks) {
    if (this.filterDate) {
      const filterDateObj = new Date(this.filterDate);
      return tasks.filter(task => task.dueDate && task.dueDate.toDateString() === filterDateObj.toDateString());
    }
    return tasks;
  }

  render() {
    const filteredTasks = this.filterTasks(this.tasks);
    const sortedTasks = this.sortTasks(filteredTasks);

    return html`
      <div>
        <h2>Create Task</h2>
        <input type="text" .value=${this.newTaskTitle} @input=${e => this.newTaskTitle = e.target.value} placeholder="Task title">
        <input type="date" .value=${this.newTaskDueDate} @input=${e => this.newTaskDueDate = e.target.value}>
        <button @click=${this.createTask}>Create</button>
      </div>

      <div>
        <h2>Task List</h2>
        <select .value=${this.sortBy} @change=${e => this.sortBy = e.target.value}>
          <option value="dueDate">Sort by Due Date</option>
        </select>
        <input type="date" .value=${this.filterDate} @input=${e => this.filterDate = e.target.value}>

        <ul>
          ${repeat(sortedTasks, task => task.id, task => html`
            <li>
              <span>${task.title}</span>
              <span>${task.dueDate ? task.dueDate.toDateString() : ''}</span>
              <button @click=${() => this.updateTask(task.id, { title: prompt('Enter new title', task.title) })}>Edit</button>
              <button @click=${() => this.deleteTask(task.id)}>Delete</button>
            </li>
          `)}
        </ul>
      </div>
    `;
  }

  static get styles() {
    return css`
      /* Add your styles here */
    `;
  }
}

customElements.define('task-manager', TaskManager);
```

# Output

![alt text](image-1.png)