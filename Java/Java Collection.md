# Index

- [Index](#index)
- [Array](#array)
  - [java.util.Arrays](#javautilarrays)
- [List](#list)
  - [java.util.ArrayList](#javautilarraylist)
- [Set](#set)
- [Map](#map)
  - [java.util.HashMap](#javautilhashmap)
    - [Use Case](#use-case)

# Array

- [Arrays - Java 8 Language Specification](https://docs.oracle.com/javase/specs/jls/se8/html/jls-10.html)

An array is a collection of items of the same type stored at contiguous memory locations.

- The size of the array is predefined and fixed.
- Java arrays are zero-indexed.

**Declare an array** - We can either assign an array to `null` or an empty array.

```java
int[] numbers = null;
int[] numbers = new int[0];
```

**Initialize an array** - We need to know the size because the JVM must reserve the contiguous memory block for it.

```java
int[] numbers = new int[100];
int[] numbers = {1, 2, 3};

int[][] matrix = new int[10][10];
int[][] matrix = {{1, 2, 3}, {4, 5, 6}, {7, 8, 9}};

// If we use the `var` keyword
var numbers = new int[100];
var numbers = new int[]{1, 2, 3, 4, 5};
```

**Size of an array** - We can use the associated variable `length`.

```java
int[] arr = {1, 2, 3};
arr.length;
```

## java.util.Arrays

- [Arrays - Java 8](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html)
- [Arrays - Java 21](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Arrays.html)

**`List<T> asList(T... a)`**

Returns a fixed-size list backed by the specified array. The returned list is implemented by `Arrays` class itself, called `Arrays$ArrayList`, which is different from a real `ArrayList`.

Changes made to the array will be visible in the returned list, and changes made to the list will be visible in the array. The list is just a view on the original array and cannot add new element.

Therefore, this method only acts as bridge between array-based and collection-based APIs.

# List

- [List - Java 8](https://docs.oracle.com/javase/8/docs/api/java/util/List.html)

**List - Interface**

- Represents an ordered sequence of values.
- Allows duplicate elements.
- Allows multiple null elements.

**ArrayList - Class**

- Backed by an array, hence data are stored in contiguous memory locations.
- Implements a `RandomAccess` interface, which means accessing any random element will be done in O(1) complexity.
- Not synchronized.

**LinkedList - Class**

- Backed by a list of nodes holding a data field and a reference of address, hence using more memory than `ArrayList`.
- Not synchronized.

**Time Complexity**

| Method                           | Description    | ArrayList    | LinkedList |
|----------------------------------|----------------|--------------|------------|
| `add(element)`                   | Add to the end | O(1) ~ O(n)* | O(1)       |
| `add(index, element)`            | Insert         | O(n)^        | O(n)^      |
| `get(index)`                     |                | O(1)         | O(n)       |
| `set(index, element)`            |                | O(1)         | O(n)       |
| `remove(index)` / `remove(obj)`  |                | O(n)^        | O(n)^      |
| `indexOf(obj)` / `contains(obj)` | Search         | O(n)         | O(n)       |
| `size()`                         |                | O(1)         | O(1)       |

*: Worst-case scenario, when a new array has to be created and all the elements copied to it, it's O(n).

^: For `ArrayList`, both insert and remove are O(n) operation because after the operation, all elements with larger indices need to be shifted. However, for `LinkedList`, they are O(n/2) operation, which simplifies to O(n), because you need to find the element first before removing it. Therefore, in general `LinkedList` inserts / removes elements faster than `ArrayList`.

## java.util.ArrayList

- [ArrayList - Java 11](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ArrayList.html)

**`boolean equals(Object o)`**

Two lists are defined to be equal if they contain the same elements in the same order.

**`int hashCode()`**

The hash code of a list is defined to be the result of the following calculation:

```java
int hashCode = 1;
for (E e : list)
    hashCode = 31*hashCode + (e==null ? 0 : e. hashCode());
```

This ensures that `list1.equals(list2)` implies that `list1.hashCode() == list2.hashCode()` for any two lists, `list1` and `list2`, as required by the general contract of `Object.hashCode`.

# Set

- [Set - Java 8](https://docs.oracle.com/javase/8/docs/api/java/util/Set.html)

**Set - Interface**

- Contains no duplicate elements.
- Contains at most one null element.

**HashSet - Class**

- Backed by a `HashMap`.
- It does not guarantee that the order will remain constant over time.
- Not synchronized.

**Time Complexity**

| Method          | HashSet |
|-----------------|---------|
| `add(element)`  | O(1)    |
| `remove(obj)`   | O(1)    |
| `contains(obj)` | O(1)    |
| `size()`        | O(1)    |

# Map

- [Map - Java 8](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html)

**Map - Interface**

- Contains key-value pairs. Every key is exactly mapped to one value and no duplicate key is allowed.

**HashMap - Class**

- Not synchronized.

**Time Complexity**

| Method            | HashMap |
|-------------------|---------|
| `get(key)`        | O(1)    |
| `put(key, value)` | O(1)    |

## java.util.HashMap

- [HashMap - Java 8](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html)

| Method of `HashMap<K,V>`          |
|-----------------------------------|
| `Collection<V> values()`          |
| `Set<K> keySet()`                 |
| `get(key)`                        |
| `getOrDefault(key, defaultValue)` |
| `put(key, value)`                 |

**`Collection<V> values()`**

Returns a `Collection` view of the values contained in this map. The collection is backed by the map, so changes to the map are reflected in the collection, and vice-versa.

**`Set<K> keySet()`**

Returns a `Set` view of the keys contained in this map. The set is backed by the map, so changes to the map are reflected in the set, and vice-versa.

### Use Case

**Update or Insert**

```java
Map<Integer, Integer> frequencyByNumber = new HashMap<>();

frequencyByNumber.put(key, frequencyByNumber.getOrDefault(key, 0) + 1)
```

**Iteration of the `HashMap`:**

```java
Map<Integer, Integer> frequencyByNumber = new HashMap<>();

// When both key and value are required
for(var number : frequencyByNumber.keySet()) {
    var frequency = frequencyByNumber.get(number);
}
```