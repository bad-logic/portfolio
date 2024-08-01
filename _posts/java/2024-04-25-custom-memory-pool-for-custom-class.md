---
layout: post
title: 'Custom Memory Pool: Creating memory pool for custom class'
date: 2024-04-25 12:12:12 -0500
categories: posts java
tags: Java,stringpoolconstant,custommemorypool#hashmap
---

In this blog we are going to see how we can implement a string pool like cache for our own custom class.

<b>Problem statement:</b>
I want to create a `Person` class that stores `name`, `ssn`, `address` and `email`. but if i create a multiple person with the same `name` and `ssn` value, it should be considered the same person and `no new object` should be created in the heap. It should `return the  memory address of the old object`. but if the combination of ssn and name is unique, it should create a new `Person` object in the heap.

<b>Data Structure:</b>
From the problem statement we can see that we need to store a key value pairs. and also the key needs to be unique representing a unique person. so we will be working with `HashMap`.

<b>Concept:</b>
We will be implementing the `string pool` like feature where if we find object in memory, we will return memory address of that object, else create a new object and store in the memory pool.

<b>Implementation:</b>

- Setting up a Person class

```java
public class Person{
  private String name;
  private String ssn;
  private String address;
  private String email;

  public Person(String name,String ssn,String address,String email){
    this.name = name;
    this.ssn = ssn;
    this.address = address;
    this.email = email;
 }
}
```

- we need a memory pool to store the Person objects

```java
   private static final HashMap<String,Person> MEMORY_POOL = new HashMap<>();
```

we are using static variable because all Person objects will be sharing this variable.

- we need a method to generate unique key combination for each Person object.

```java
private static int generateKey(String name, String ssn){
  // simple key,name combination would also work
  // but you would need to change Integer to String in hashmap
  return Objects.hash(name,ssn);
}
```

- Next we will need a method to store Person object in the memory pool.

```java
private static void addToPool(Integer key, Person value ){
  Person.MEMORY_POOL.putIfAbsent(key,value);
}
```

- Since constructor call always initializes new object we need to call constructor conditionally, and also we do not want anyone to call constructor directly. so making constructor private and exposing static method to create Person object, this method makes a decision to either create a new object or get from the memory pool.

```java

private Person(String name,String ssn,String address,String email){
   this.name = name;
   this.ssn = ssn;
   this.address = address;
   this.email = email;
}
public static Person createPerson(String name,String ssn,String address,String email){
  int key = Person.generateKey(name,ssn);
  if(Person.MEMORY_POOL.containsKey(key)){
    return MEMORY_POOL.get(key);
  }
  Person p = new Person(name,ssn,address,email);
  Person.addToPool(key,p);
 return p;
}

```

- Putting it all together.

```java
import java.util.HashMap;
import java.util.Objects;

public class Person{
    private static final HashMap<Integer,Person> MEMORY_POOL =
            new HashMap<>();

    private String name;
    private String ssn;
    private String address;
    private String email;

    private Person(String name,String ssn,String address,String email){
        this.name = name;
        this.ssn = ssn;
        this.address = address;
        this.email = email;
    }

    private static int generateKey(String name, String ssn){
        return Objects.hash(name,ssn);
    }

    private static void addToPool(Integer key, Person value ){
        Person.MEMORY_POOL.putIfAbsent(key,value);
    }

    public static Person createPerson(String name,String ssn,String address,String email){
        int key = Person.generateKey(name,ssn);
        if(Person.MEMORY_POOL.containsKey(key)){
            return MEMORY_POOL.get(key);
        }
        Person p = new Person(name,ssn,address,email);
        Person.addToPool(key,p);
        return p;
    }

    private String getName(){
        return this.name;
    }
    public String getEmail(){
        return this.email;
    }
    public String getAddress(){
        return this.address;
    }

    public void setEmail(String email){
        this.email = email;
    }

    public void setAddress(String address){
        this.address = address;
    }

    public String toString(){
        return "Person ( name: "+ this.name + ", ssn: " + this.ssn + ", email: " + this.email + ", address: "+this.address + " )";
    }

}
```

- Testing

```java
public class Main {

    public static void main(String[] args){
        Person p = Person.createPerson("john","234-df-44","usa", "john@usa.email.com");
        System.out.println("Person p: " + p);

        // should not create a Person object since name and ssn are same
        Person p1 = Person.createPerson("john","234-df-44","mexico", "john@mexico.email.com");
        System.out.println("Person p1: " + p1);

        Person p2 = Person.createPerson("Johnny","776-dr4-990","canada", "johnny@canada.email.com");
        System.out.println("Person p2: " + p2);

        System.out.println();
        // checking if the p and p1 reference to the same memory location
        System.out.println("p==p1: " + (p == p1)); // true
        // can also be verified with equals method since
        // we never override the method in Person class
        System.out.println("p.equals(p1): " + p.equals(p1)); // true

        // checking if p1 and p2 are equal
        System.out.println("p1==p2: " + (p1 == p2)); // false

        System.out.println();
        System.out.println("updating the email for p1 to hello@world.com");
        p1.setEmail("hello@world.com");
        System.out.println("Person p: " + p);
        System.out.println("Person p1: " + p1);
    }
}

```

On running the above main method we can see the output as:

```bash
Person p: Person ( name: john, ssn: 234-df-44, email: john@usa.email.com, address: usa )
Person p1: Person ( name: john, ssn: 234-df-44, email: john@usa.email.com, address: usa )
Person p2: Person ( name: Johnny, ssn: 776-dr4-990, email: johnny@canada.email.com, address: canada )

p==p1: true
p.equals(p1): true
p1==p2: false

updating the email for p1 to hello@world.com
Person p: Person ( name: john, ssn: 234-df-44, email: hello@world.com, address: usa )
Person p1: Person ( name: john, ssn: 234-df-44, email: hello@world.com, address: usa )
```

From the output we can see that `p and p1` are same objects. since they have same `ssn and name` combination, new Person object `("john","234-df-44","mexico","john@mexico.email.com")` is not created. The email update of `p1` is reflected on both `p1 and p object`, which shows that they both point to the `same object` in the heap.
