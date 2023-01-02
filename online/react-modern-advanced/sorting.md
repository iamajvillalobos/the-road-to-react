# Real World React (Advanced)

We've covered most of React's fundamentals, its legacy features, and techniques for maintaining applications. Now it's time to dive into developing real-world React features. Each of the following sections will come with a task. Try to tackle these tasks without the *optional hints* first, but be aware that these are going to be challenging on your first attempt. If you need help, use the *optional hints* or follow the instructions from the section.

## Sorting

**Task:** Working with a list of items often includes interactions that make data more approachable by users. So far, every item was listed with each of its properties. To make it explorable, the list should enable the sorting of each property by title, author, comments, and points in ascending or descending order. Sorting in only one direction is fine, because sorting in the other direction will be part of the next task.

**Optional Hints:**

* Introduce a new sort state in the App or List component.
* For each property (e.g. `title`, `author`, `points`, `num_comments`) implement an HTML button which sets the sort state for this property.
* Use the sort state to apply an appropriate sort function on the `list`.
* Using a utility library like [Lodash](https://lodash.com) for its `sortBy` function is encouraged.

![](images/sort.png)

Okay, let's tackle this task! We will treat the list of data like a table. Each row represents an item of the list and each column represents one property of the item. Introducing headers should provide the user more guidance about each column:

```javascript{3-9}
const List = ({ list, onRemoveItem }) => (
  <ul>
    <li style={{ display: 'flex' }}>
      <span style={{ width: '40%' }}>Title</span>
      <span style={{ width: '30%' }}>Author</span>
      <span style={{ width: '10%' }}>Comments</span>
      <span style={{ width: '10%' }}>Points</span>
      <span style={{ width: '10%' }}>Actions</span>
    </li>

    {list.map((item) => (
      <Item
        key={item.objectID}
        item={item}
        onRemoveItem={onRemoveItem}
      />
    ))}
  </ul>
);
```

We are using inline style for the most basic layout. To match the layout of the header with the rows, give the rows in the Item component a layout as well:

```javascript{2-3,6-9}
const Item = ({ item, onRemoveItem }) => (
  <li style={{ display: 'flex' }}>
    <span style={{ width: '40%' }}>
      <a href={item.url}>{item.title}</a>
    </span>
    <span style={{ width: '30%' }}>{item.author}</span>
    <span style={{ width: '10%' }}>{item.num_comments}</span>
    <span style={{ width: '10%' }}>{item.points}</span>
    <span style={{ width: '10%' }}>
      <button type="button" onClick={() => onRemoveItem(item)}>
        Dismiss
      </button>
    </span>
  </li>
);
```

In the ongoing implementation, we will remove the style attributes, because it takes up lots of space and clutters the actual implementation logic (hence extracting it into proper CSS). But I encourage you to keep it for yourself.

The List component will handle the new sort state. This can also be done in the App component, but in the end, only the List component needs it, so we can lift the state directly to it. The sort state initializes with a `'NONE'` state, so the list items are displayed in the order they are fetched from the API. Furthermore, we will add a new handler to set the sort state with a sort-specific key:

```javascript{1-8,10-11}
const List = ({ list, onRemoveItem }) => {
  const [sort, setSort] = React.useState('NONE');

  const handleSort = (sortKey) => {
    setSort(sortKey);
  };

  return (
    ...
  );
};
```

In the List component's header, buttons can help us to set the sort state for each column/property. An inline handler is used to sneak in the sort-specific key (`sortKey`). When the button for the "Title" column is clicked, `'TITLE'` becomes the new sort state:

```javascript{8-10,13-15,18-20,23-25}
const List = ({ list, onRemoveItem }) => {
  ...

  return (
    <ul>
      <li>
        <span>
          <button type="button" onClick={() => handleSort('TITLE')}>
            Title
          </button>
        </span>
        <span>
          <button type="button" onClick={() => handleSort('AUTHOR')}>
            Author
          </button>
        </span>
        <span>
          <button type="button" onClick={() => handleSort('COMMENT')}>
            Comments
          </button>
        </span>
        <span>
          <button type="button" onClick={() => handleSort('POINT')}>
            Points
          </button>
        </span>
        <span>Actions</span>
      </li>

      {list.map((item) => ... )}
    </ul>
  );
};
```

The state management for the new feature is implemented, but we don't see anything when our buttons are clicked yet. This happens because the sorting mechanism hasn't been applied to the actual `list`. Sorting an array with JavaScript isn't trivial, because every JavaScript primitive (e.g. string, boolean, number) comes with edge cases when an array is sorted by its properties. We will use a library called [Lodash](https://lodash.com) to solve this, which comes with many JavaScript utility functions (e.g. `sortBy`). First, install it via the command line:

```text
npm install lodash
```

Second, at the top of your file, import the utility function for sorting:

```javascript{3}
import * as React from 'react';
import axios from 'axios';
import { sortBy } from 'lodash';

...
```

Third, create a JavaScript object (also called dictionary in this case) with all the possible `sortKey` and sort function mappings. Each specific sort key is mapped to a function that sorts the incoming `list`. Sorting by `'NONE'` returns the unsorted list; sorting by `'POINT'` returns a list and its items sorted by the `points` property, and so on:

```javascript{1-7}
const SORTS = {
  NONE: (list) => list,
  TITLE: (list) => sortBy(list, 'title'),
  AUTHOR: (list) => sortBy(list, 'author'),
  COMMENT: (list) => sortBy(list, 'num_comments').reverse(),
  POINT: (list) => sortBy(list, 'points').reverse(),
};

const List = ({ list, onRemoveItem }) => {
  ...
};
```

With the `sort` (`sortKey`) state and all possible sort variations (`SORTS`) at our disposal, we can sort the list before mapping it:

```javascript{8-9,15}
const List = ({ list, onRemoveItem }) => {
  const [sort, setSort] = React.useState('NONE');

  const handleSort = (sortKey) => {
    setSort(sortKey);
  };

  const sortFunction = SORTS[sort];
  const sortedList = sortFunction(list);

  return (
    <ul>
      ...

      {sortedList.map((item) => (
        <Item
          key={item.objectID}
          item={item}
          onRemoveItem={onRemoveItem}
        />
      ))}
    </ul>
  );
};
```

Task's done and here comes the recap: First we extracted the sort function from the dictionary by its `sortKey` (state), then we applied the function to the list before mapping it to render each Item component. Second, we rendered HTML buttons as header columns to give our users interaction. Then, we added implementation details for each button by changing the sort state. Finally, we used the sort state to sort the actual list.

### Exercises:

* Compare your source code against the author's [source code](https://bit.ly/3LN8IhE).
  * Recap all the [source code changes from this section](https://bit.ly/3BN0m58).
* Read more about [Lodash](https://lodash.com).
* Why did we use numeric properties like `points` and `num_comments` for a reverse sort?
* Use your styling skills to give the user feedback about the current active sort. This mechanism can be as straightforward as giving the active sort button a different color.
* Optional: [Leave feedback for this section](https://forms.gle/GM71SDZZWPQWEwmB7).
