---
layout: post
title: 'Understanding Java’s Pass by Value with Heap and Stack Frames Visualization'
date: 2024-04-24 21:02:44 -0500
categories: posts java
tags: Java,PassByValue,ReferenceOfObjectsByValue
cover_image: /assets/images/understanding_pass_by_value_with_heap_stack_frames/heap_stack_visualization_jbi4qsvnpuqlkw8lss0j.avif
---

Java is well-known for being strong and flexible, but it can be confusing, especially about how it handles passing things to methods. People often wonder if Java passes things by value or by reference. In this blog, we’ll talk about this and see how things go down when object arguments are passed.

let’s first understand what is pass by value and pass by reference.

<b>Pass by value:</b> When arguments are provided to a method, a copy of those arguments are passed to the method. Any modification made to that parameter inside the method’s scope, has no affect on the original variable outside the method’s scope.

<b>Pass by Reference:</b> When a reference or pointer to the original variables are available inside a methods scope, then it is pass by reference. Any modification made will also be reflected to the original variable.

Java strictly follows `Pass By Value` , the misconception generally occurs when passing objects as arguments.

Let’s look at an example to clear things up.

```java
public class Employee{

 private String name;
 private double salary;

 Employee(String n){
  this.name = n;
 }

 public String getName(){
  return this.name;
 }

 public double getSalary(){
  return this.salary;
 }

 public void setSalary(double s){
  this.salary = s;
 }

 public String toString(){
  return "Employee: name: " + this.getName() + ", salary: "+ this.getSalary();
 }

}
```

```java
public class Main{

 public static void swapValues(Employee e1,Employee e2){
  Employee temp = e1;
  e1 = e2;
  e2 = temp;
 }

 public static void changeValue(Employee e1){
  e1.setSalary(3400.99);
  e1 = new Employee("Ben");
  e1.setSalary(4500.88);
 }

 public static void main(String[] args){
  Employee e1 = new Employee("Bar");
  Employee e2 = new Employee("Foo");

  System.out.println("e1 before swap: "+e1);
  System.out.println("e2 before swap: "+e2);

  swapValues(e1,e2);

  System.out.println("e1 after swap: "+e1);
  System.out.println("e2 after swap: "+e2);

  changeValue(e1);
  System.out.println("e1 after changing value: "+e1);
 }

}

```

when you run the Main class the following output can be seen:

```bash
e1 before swap: Employee: name: Bar, salary: 0.0
e2 before swap: Employee: name: Foo, salary: 0.0
e1 after swap: Employee: name: Bar, salary: 0.0
e2 after swap: Employee: name: Foo, salary: 0.0
e1 after changing value: Employee: name: Bar, salary: 3400.99
```

Let's dig deeper and understand how things go down in the `swapValues` and `changeValue` methods. let's look at the main method.

whenever a `new` operator is used to create an object, a memory address is created on a heap for that object and its memory address is returned.
In the example above `e1` and `e2` store an address of the objects memory location in heap.

> <i>let us assume the memory address for e1 is 50 and e2 is 100.</i>

![pass_by_value_heap_stack_illustration](/assets/images/understanding_pass_by_value_with_heap_stack_frames/heap_stack_visualization_jbi4qsvnpuqlkw8lss0j.avif)

when `e1` and `e2` are passed to the swapValues method, instead of passing the original value, two new Employee variables are created and passed to the method. These new variables are a copy pointing to the same memory location in heap as the original variables.

![swapValues_stack_heap_illustration](/assets/images/understanding_pass_by_value_with_heap_stack_frames/heap_stack_visualization_1yjgbi4wikcgko36u7yk.avif)

when you are performing operations on these variables inside the swapValues method you are actually operating on the copy values.

![swap_heap_stack_illustration](/assets/images/understanding_pass_by_value_with_heap_stack_frames/heap_stack_visualization_cbofywa98o5v1bx8u6io.avif)

In above image we can see that e1 and e2 ( local to the swapValues ) have swapped the values but since java is pass by value, the original e1 and e2 does not reflect those changes. After the execution of the method completes the original variables remain unaffected.

![pass_by_value_heap_stack_illustration](/assets/images/understanding_pass_by_value_with_heap_stack_frames/heap_stack_visualization_jbi4qsvnpuqlkw8lss0j.avif)

let’s look at the `changeValue` method. Similar to swapValues method, a new Employee variable pointing to the same memory address as e1 is created and passed to the changeValue method.

![change_value_stack_heap_illustration](/assets/images/understanding_pass_by_value_with_heap_stack_frames/heap_stack_visualization_t16ue5pa249f5pol3mma.avif)

now when i call `e1.setSalary(3400.99)` , you are performing set operation on the object stored at memory location 50. so this change will be reflected on both e1, as they point to the same object in heap.

![setSalary_stack_heap_illustration](/assets/images/understanding_pass_by_value_with_heap_stack_frames/heap_stack_visualization_bw7en8tbnaeijwpg157t.avif)

Any reassignment to the local variable will be bound to that scope only. The reassignment `e1=new Employee("Ben”);` will create a new object in the heap and store the new memory location.

![e1_reassignment_stack_heap_illustration](/assets/images/understanding_pass_by_value_with_heap_stack_frames/heap_stack_visualization_psybo2u389rub0e05kv2.avif)

Moving forward any update methods on e1 will be reflected only on the new object at location 150. `e1.setSalary(4500.88);` can be seen on the stack and heap as follows:

![e1_object_setSalary_heap_stack_illustration](/assets/images/understanding_pass_by_value_with_heap_stack_frames/heap_stack_visualization_kw3nzzeghu2ygith5dnx.avif)

After the changeValue’s method completes execution, we can see, only the salary of the original employee e1 is updated. The new Employee object assignment has no effect on the main’s e1.

![execution_completion_stack_heap_illustration](/assets/images/understanding_pass_by_value_with_heap_stack_frames/heap_stack_visualization_nlb7crddkkov5yragrlr.avif)

we saw that any operation to the variables, had no affect to the original variables. Although we could see that setting the salary on the e1 object worked, this is only because we operated on the same object at memory location 50, but <b><i>not because of pass by reference</i></b>. Even when working with objects we can see that the objects are <b><i>not passed by reference rather reference to the objects are passed by value.</i></b>
