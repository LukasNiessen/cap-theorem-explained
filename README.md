# ELI5: CAP Theorem in System Design

This is a super simple ELI5 explanation of the CAP Theorem. After that, I explain a common misunderstanding that you should be careful of, and then lastly, I will give two system design examples where CAP Theorem is used to make design decision.

## Super simple explanation

C = Consistency = Every user gets the same data  
A = Availability = Users can retrieve the data always  
P = Partition tolerance = Even if there are network issues, everything works fine still

Now the CAP Theorem states that in a distributed system, you need to decide whether you want consistency or availability. You cannot have both.

### Questions

**And in non-distributed systems?** CAP Theorem only applies to distributed systems. If you only have one database, you can totally have both. (Unless that DB server if down obviously, then you have neither.

**Is this always the case?** No, if everything is working fine, then we have both, consistency and availability. However, if a server looses internet access for example, or there is any other fault that occurs, THEN we have only one of the two, that is either have consistency or availability.

### Example

As I said already, the problems only arises, when we have some sort of fault. Let's look at this example.

```
    US (Master)                    Europe (Replica)
   ┌─────────────┐                ┌─────────────┐
   │             │                │             │
   │  Database   │◄──────────────►│  Database   │
   │   Master    │    Network     │   Replica   │
   │             │  Replication   │             │
   └─────────────┘                └─────────────┘
        │                              │
        │                              │
        ▼                              ▼
   [US Users]                     [EU Users]
```

**Normal operation:** Everything works fine. US users write to master, changes replicate to Europe, EU users read consistent data.

**Network partition happens:** The connection between US and Europe breaks.

```
    US (Master)                    Europe (Replica)
   ┌─────────────┐                ┌─────────────┐
   │             │    ╳╳╳╳╳╳╳     │             │
   │  Database   │◄────╳╳╳╳╳─────►│  Database   │
   │   Master    │    ╳╳╳╳╳╳╳     │   Replica   │
   │             │    Network     │             │
   └─────────────┘     Fault      └─────────────┘
        │                              │
        │                              │
        ▼                              ▼
   [US Users]                     [EU Users]
```

Now we have two choices:

**Choice 1: Prioritize Consistency (CP)**

- EU users get error messages: "Database unavailable"
- Only US users can access the system
- Data stays consistent but availability is lost for EU users

**Choice 2: Prioritize Availability (AP)**

- EU users can still read/write to the EU replica
- US users continue using the US master
- Both regions work, but data becomes inconsistent (EU might have old data)

## What are Network Partitions?

Network partitions are when parts of your distributed system can't talk to each other. Think of it like this:

- Your servers are like people in different rooms
- Network partitions are like the doors between rooms getting stuck
- People in each room can still talk to each other, but can't communicate with other rooms

Common causes:

- Internet connection failures
- Router crashes
- Cable cuts
- Data center outages
- Firewall issues

The key thing is: **partitions WILL happen**. It's not a matter of if, but when.

## The "2 out of 3" Misunderstanding

CAP Theorem is often presented as "pick 2 out of 3." This is wrong.

**Partition tolerance is not optional.** In distributed systems, network partitions will happen. You can't choose to "not have" partitions - they're a fact of life, like rain or traffic jams... :-)

So our choice is: **When a partition happens, do you want Consistency OR Availability?**

- **CP Systems:** When a partition occurs → node stops responding to maintain consistency
- **AP Systems:** When a partition occurs → node keeps responding but users may get inconsistent data

In other words, it's not "pick 2 out of 3," it's "partitions will happen, so pick C or A."

## System Design Example 1: Social Media Feed

**Scenario:** Building Netflix

**Decision:** Prioritize Availability (AP)

**Why?** If some users see slightly outdated movie names for a few seconds, it's not a big deal. But if the users cannot watch movies at all, they will be very unhappy.

## System Design Example 2: Flight Booking System

In here, we will not apply CAP Theorem to the entire system but to parts of the system. So we have two different parts with different priorities:

### Part 1: Flight Search

**Scenario:** Users browsing and searching for flights

**Decision:** Prioritize Availability

**Why?** Users want to browse flights even if prices/availability might be slightly outdated. Better to show approximate results than no results.

### Part 2: Flight Booking

**Scenario:** User actually purchasing a ticket

**Decision:** Prioritize Consistency

**Why?** If we would prioritize availibility here, we might sell the same seat to two different users. Very bad. We need strong consistency here.

### PS: Architectural Quantum

What I just described, having two different _scopes_, is the concept of having more than one _architecture quantum_. There is a lot of interesting stuff online to read about the concept of architecture quanta :-)

# Feedback

Feel free to contribute by submitting a PR or creating an issue.  
**If this was helpful, you can show support by giving this repository a star! 🌟😊**
