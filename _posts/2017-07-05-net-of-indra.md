---
layout: post
title: The Net of Indra in JavaScript
---

>Far away in the heavenly abode of the great god Indra, there is a wonderful net which has been hung by some cunning artificer in such a manner that it stretches out infinitely in all directions. In accordance with the extravagant tastes of deities, the artificer has hung a single glittering jewel in each "eye" of the net, and since the net itself is infinite in dimension, the jewels are infinite in number. There hang the jewels, glittering "like" stars in the first magnitude, a wonderful sight to behold. If we now arbitrarily select one of these jewels for inspection and look closely at it, we will discover that in its polished surface there are reflected all the other jewels in the net, infinite in number. Not only that, but each of the jewels reflected in this one jewel is also reflecting all the other jewels, so that there is an infinite reflecting process occurring.

![https://thesupremescience.files.wordpress.com/2014/11/4183494604_e56101e4d0_o.jpg](The Net of Indra)

The metaphor of Indra's net illustrates the concepts of `emptiness` and `interpenetration` in Buddhism. The idea is wonderfully puzzling and aesthetically appealing. [Graham Priest](https://www.google.com/search?q=graham+priest&oq=graham+priest&aqs=chrome..69i57j69i60j69i61l2j69i60j35i39.2254j0j4&sourceid=chrome&ie=UTF-8) has at least one very article on the topic, formalizing the idea with graph theory ([Google Books](https://books.google.com/books?id=MMX9CAAAQBAJ&printsec=frontcover&source=gbs_atb#v=onepage&q=indra&f=false)). In this post I want to try capture the idea in JavaScript. It won't be as precise, but should push us closer to the precision of formal graph theory while retaining the character of the worldview. If you happen to like JavaScript, metaphysics, and Buddhism you've arrived -- if not, this post may not make much sense.

As a first pass, let's say that `emptiness` is the opposite of independent existence. From Nagurjuna:

> By their nature, things are not a determinate entity. Their nature is a non-nature; it is their non-nature that is their nature. For they have only one nature, i.e., no nature.

Let `interpenetration` be the idea from Priest: "To grasp one is, in some sense, to grasp all." It is not merely an epistemic idea, but the idea that because you what one object is you must grasp what the others are due to their interpenetrated nature.

This may be slightly opaque.

Let's take a left turn into something simple, an ordinary `LinkedListNode` class:

```javascript
function LinkedListNode(value) {
  this.value = value;
  this.next = null;
}
```

We could create a LinkedListNode as follows:

```javascript
let ordinaryNode = new LinkedListNode("ordinary")

console.log(node) // LinkedListNode { value: 'ordinary', next: null }
```

In a sense our `ordinaryNode` has independent existence. It carries a value, its name: `ordinary`. We can set its next pointer to some other node and it would still carry that value. In fact we can create an entire list of nodes:

```javascript
let nextNode = new LinkedListNode("next node"),
  nextNextNode = new LinkedListNode("next next node");

ordinaryNode.next = nextNode;
nextNode = nextNextNode;
```

`ordinaryNode` is now the head of the list which we can represent as follows:

```javascript
"ordinary" => "next node" => "next next node"
```

This does not look like our net. Every node in the list has its own existence, so it cannot represent emptiness at all. We can grasp `next node` without grasping `ordinary` which can be seen if we log `next node`:

```javascript
console.log(nextNode) // LinkedListNode { value: 'ordinary', next: [Object] }
```

Every node does encode every other node, hence a singly linked list hardly represents interpenetration.

A graph is a better representation of Indra's net. Let's call every edge in the graph a relation:

```javascript
function Relation(...relata) {
  this.relata = relata;
}
```

Every instance of `Relation` takes in relatum (the objects related) as relata
and sets them to the instance variable `relata`.

We could create a Node class as follows:

```javascript
function Node() {
  this.relations = [];
}

Node.validRelatum = function(relatum) {
  if (!relatum instanceof Node) {
    throw new Error("Every relatum must be a instance of Node");
  }
}

Node.addRelation = function(relation, ...relata) {
  for(let relatum of relata) {
    relatum.relations.push(relation);
  }
}

Node.prototype.createRelation = function(relatum, type) {
  Node.validRelatum(relatum);

  let relation = new Relation(type, this, relatum);

  Node.addRelation(relation, this, relatum)
}
```

Essentially, we can create Nodes and the relations between them. For example:

```javascript
let a = new Node(),
    b = new Node(),
    c = new Node();

a.createRelation(b)
b.createRelation(c)
console.log(b) /*
Node {
  relations: [
    Relation {
      relata: [Object]
    }, Relation {
      relata: [Object]
    }
  ]
}
*/
```

We're getting closer. Now `b` (and `a` and `c`) is just a node that contains a list of relations to other nodes. What it is to be `b` is to be related to `a` and `c` in various ways. And what it is to be `a` and `c` is to be related to other nodes and so on for any other nodes we may add. They lack values of their own, in that sense they are empty. Moreover, when we "grasp" any node, like `c`, we have access to every other node in the web. We can grasp `a` though the relation that `c` has to `b`. Hence the nodes are in some sense interpenetrated.

The problem is that `a`, `b`, and `c` are instances of the class `Node`. To make the JavaScript representation more aligned with the net of Indra, we should have every relata be an instance of `Relation`. We want to be able to log something like this:

```JavaScript
/*
Relation {
  relata: [
    Relation {
      relata: [Object]
    }, Relation {
      relata: [Object]
    }
  ]
}
*/
```
Here, all things objects are composed of `Relations` and nothing more. Moreover, every instance of a `Relation` contains every other instance in the sense that every relation can be eventually be reached through traversing the `relata` properties.

Ok, so just what is this JS representing? The relational nature of all things. [Wat](https://www.destroyallsoftware.com/talks/wat)?

![http://i2.cdn.cnn.com/cnnnext/dam/assets/130502152627-rubber-duck-in-hong-kong-1-horizontal-large-gallery.jpg](Wat duck)

Priest give the helpful example of time. Newton thought that time had its own independent existence. The time, 1066, could have existed without any other thing in the universe existing. There are possible worlds which only include a few times and nothing else. On the other hand, for the German philosopher Leibniz, what it is to be a time is a purely relational matter. The time 1066 came after the reign of Marcus Aurelius and before the Renaissance. Time is defined by its relation to other things. This seems much more reasonable to me.

Now just extend that picture to everything! Priest continues:

> What is it to be Graham Priest? My being is constituted by having born in London in 1948, being the child of George and Laura, residing most of my adult life in Australia, being the father of Marcus and Annika, dying in ?, and so on... I am essentially constituted by my place in that web of relations

In that sense, everything is empty. Priest analyzes interpenetration in terms of a tree. For instance, take the north pole, `n`, and the south pole, `s`. We can represent it as a tree as follows:

```text
      n
   /  | \
  .   s  .
  .   |  .
  .   .  .
```

But of course, what it is to be `n` is to be related to `s` in a particular way, so we should extend the tree:

```text
      n
   /  |  \
  .   s   .
  .   |   .
  .   n   .
   /  |  \
  .   .   .
  .   .   .
  .   .   .
```

Interpenetration is symmetric, so just as `n` contains `s` as a subtree, `s` contains `n` as a subtree.

Priest states that:

> One could not hope for a nicer representation of the idea that two objects "interpenetrate"

This isn't so clear to me. Though Priest's the infinitely branching picture is appealing, I don't see why a the circular representation isn't at least as good. In this representation, each `Relation` continues to encode on another as they contain reference to each other in `relata`. This is what the representation in JavaScript adds, the release from infinite trees.

Do check out Priest's essay. It becomes much more profound. I've talked about emptiness and interpenetration here, but haven't begun to touch on the (conventional) distinction between conventional and ultimate reality.
