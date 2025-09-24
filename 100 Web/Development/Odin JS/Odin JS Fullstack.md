**DO NOT SKIP ANYTHING!**
![[Pasted image 20250919201047.png]]
https://xyproblem.info/
https://dontasktoask.com/
https://www.theodinproject.com/guides/community/how_to_ask

# JavaScript Array Methods - Definitions and Tips

## Core Array Methods

### 1. **Filter** 
**Definition**: Creates a new array with all elements that pass a test (provided as a function)

```javascript
// Basic syntax
const filtered = array.filter(item => condition);

// Example: Filter inventors born in 1500s
const fifteen = inventors.filter(inventor => 
  inventor.year >= 1500 && inventor.year < 1600
);
```
**What it does**: Takes each element, applies a conditional statement. If true → element goes in new array. If false → element is excluded.

### 2. **Map**
**Definition**: Creates a new array by transforming each element using a provided function

```javascript
// Basic syntax  
const mapped = array.map(item => transformation);

// Example: Create array of full names
const fullNames = inventors.map(inventor => 
  `${inventor.first} ${inventor.last}`
);
```
**What it does**: Takes each element, transforms it according to your function, returns new array with transformed elements.

### 3. **Sort**
**Definition**: Sorts array elements in place and returns the sorted array

```javascript
// Basic syntax
const sorted = array.sort((a, b) => comparison);

// Example: Sort by birth year (oldest first)
const ordered = inventors.sort((a, b) => a.year - b.year);

// Example: Sort by years lived (longest lived first)
const oldest = inventors.sort((a, b) => 
  (b.passed - b.year) - (a.passed - a.year)
);
```
**What it does**: Compares pairs of elements. Return negative (a before b), positive (b before a), or zero (equal).

### 4. **Reduce**
**Definition**: Reduces array to a single value by executing a function for each element

```javascript
// Basic syntax
const result = array.reduce((accumulator, current) => operation, initialValue);

// Example: Calculate total years lived by all inventors
const totalYears = inventors.reduce((total, inventor) => {
  return total + (inventor.passed - inventor.year);
}, 0);
```
**What it does**: Goes through each element, accumulates a result. Can return anything - number, string, object, or array.

## Speaker's Key Tips

### **Progressive Code Refinement**
- Start with verbose function syntax, then refactor to arrow functions
- Make code more concise without sacrificing readability

```javascript
// Verbose version
const fifteen = inventors.filter(function(inventor) {
  if (inventor.year >= 1500 && inventor.year < 1600) {
    return true;
  }
});

// Refined arrow function
const fifteen = inventors.filter(inventor => 
  inventor.year >= 1500 && inventor.year < 1600
);
```

### **console.table() for Better Debugging**
```javascript
// Instead of console.log()
console.table(inventors);
// Shows data in clean, readable table format
```

### **Sorting Shortcuts**
```javascript
// For alphabetical sorting (when last name comes first)
people.sort(); // Uses default ASCII sorting

// For numerical sorting
numbers.sort((a, b) => a - b); // Ascending
numbers.sort((a, b) => b - a); // Descending
```

### **Method Chaining**
```javascript
// Combine methods for powerful data manipulation
const result = inventors
  .filter(inventor => inventor.year >= 1500)
  .map(inventor => `${inventor.first} ${inventor.last}`)
  .sort();
```

## Important Concepts

### **Immutability**
- These methods **don't modify** the original array
- They **return new arrays** or values
- Original data stays unchanged

### **Return Values Matter**
- **Filter/Map/Sort**: Return new arrays
- **Reduce**: Returns single value (number, string, object, etc.)
- Always assign results to variables or chain methods

### **Arrow Function Benefits**
- More concise syntax
- Implicit return for single expressions
- Cleaner, more readable code