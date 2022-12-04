---
layout: post
title: Advent of Code 2022 - Day 04
---

# Day 4

The mission of today is to find overlaps in the assignments of work for pairs of elves.

[See the problem][challenge]

## Approach

Each assignment is an interval of sequential tasks. To find whether two assignment overlaps we can use the interval's
boundaries.

### Parsing

A pair of assignments is represented as `2-4,6-8` which can be parsed into a `record`. First the string is split by `,`
into two string representing a range. Then each range is split by `-` into bounds of the range.

````java
record ElfPairAssignment(int oneElfFrom, int oneElfTo, int anotherElfFrom, int anotherElfTo) {
    static ElfPairAssignment parse(String pair) {
        var assignments = pair.split(",");
        var oneElfBounds = elfBounds(assignments[0]);
        var anotherElfBounds = elfBounds(assignments[1]);

        return new ElfPairAssignment(oneElfBounds[0], oneElfBounds[1], anotherElfBounds[0], anotherElfBounds[1]);
    }

    private static int[] elfBounds(String assigment) {
        return Arrays.stream(assigment.split("-")).mapToInt(Integer::parseInt).toArray();
    }
}
````

### Full overlap

Now that we have a data structure we can calculate whether the two ranges overlap fully.

```java
private boolean fullOverlap(ElfPairAssignment pair){
        // left one contains right one
        return(pair.oneElfFrom<=pair.anotherElfFrom && pair.oneElfTo>=pair.anotherElfTo)
        // or right one contains left one
        ||(pair.anotherElfFrom<=pair.oneElfFrom && pair.anotherElfTo>=pair.oneElfTo);
        }
```

### Partial overlap

For partial overlap the formula varies slightly. It is sufficient by having one of the boundaries of the first range
within the boundaries of the second.

```java
private boolean overlap(ElfPairAssignment pair){
    // if one ends up in another
    return(pair.oneElfTo>=pair.anotherElfFrom && pair.oneElfTo<=pair.anotherElfTo)
    // if another ends up in one
    ||(pair.anotherElfTo>=pair.oneElfFrom && pair.anotherElfTo<=pair.oneElfTo);
    }
```

### Counting

Both `fullOverlap` and `overlap` can be used as `Predicate` to filter a list of assigments pairs.

```java
public long countFullOverlaps(List<String> pairs){
    return count(pairs,this::fullOverlap);
    }

public long countOverlaps(List<String> assignments){
        return count(assignments,this::overlap);
        }
private long count(List<String> pairs,Predicate<ElfPairAssignment> overlapPredicate){
        return pairs.stream()
        .map(ElfPairAssignment::parse)
        .filter(overlapPredicate)
        .count();
        }
```

### Full solution

Only thing left is to load the input file as a `List<String>` and pass it to the count methods.

```java
List<String> assignments=Files.readAllLines(Path.of(args[0]));
System.out.printf("There are %s full overlapping pairs in the file\n",finder.countFullOverlaps(assignments));
System.out.printf("There are %s simple overlapping pairs in the file\n",finder.countOverlaps(assignments));
```

[challenge]:https://adventofcode.com/2022/day/4
