# High Level State Management

The main job of React is to take our application state and turn it into DOM nodes.

#### Write out a blueprint for how the system is supposed to work before we implement it.

### Many Kinds of State

**Model Data:** The nouns in our application. *List of products, description of event, todo on a todo list, etc.*

**View/UI State**: Are those nouns sorted in ascending or descending order? *We have a list of your items. Are we filtering it? What order do we display it?*

**Session State**: Is the user even logged in?

**Communication**: Are we in the process of fetching the nouns from the server? *Are we fetching data? Are we uploading it? Was there an error?*

**Location**: Where are we in the application? Which nouns are we looking at?

### The Two Basic Kinds of State

We can reduce the above list of five kinds of state into two basic kinds:

**Model State**: This is likely the data in our application. This could be the items in a given list.

**Ephemeral State**: Stuff like the value of an input field that will be wiped away when we hit "enter". This could also be the order in which a given list is sorted.