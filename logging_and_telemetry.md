# Logging and Telemetry: A Quick Intro

Logging and telemetry are two related tools that give you a way to answer questions about how your application is working. 
Telemetry is the process of recording information on timing, success rates, usage patterns, and other information that can give a statistical picture of how well your application is performing in the aggregate, while logging is the process of recording details of individual operations that can be used to investigate when something goes wrong.

Let’s look at the difference, and what it means for your code.

## Logging

Logging is a diagnostic tool. It lets you find where your code is doing something wrong. There are two kinds of logging: failure logging and progress logging. Both kinds of logging are useful when attempting to diagnose and fix problems in your code. 

The audience for logging is engineers (including yourself) who are trying to understand your code and, in particular, trying to fix
bugs. Always keep your audience in mind when adding logging to your code.

### Failure Logging

Failure logging tells you when something has gone wrong. Maybe an exception occurred while writing to a file, or your application couldn’t connect to a database. In these cases, you should log a message indicating what went wrong.

This snippet shows an error message that is logged when a database connection attempt fails.

```java
try {
    conn = db.connect(dbUrl);
}
catch (DbConnectionException exp) {
    log.error("Failed to connect", exp);
}
```

### Progress logging

Progress logging tells you that, so far, things are going right. Use progress logging to help understand the context for a logged failure message.

This snippet adds more information about which database we're attempting to connect to, as well as the status
of the connection. This information is giving us more insight into what the code was doing when it executed,
which can be helpful when attempting to analyze a later failure.

```java
try {
    log.info("Connecting to database "+dbUrl);
    conn = db.connect(dbUrl);
    log.info("Connection successful, status: " + conn.getStatus());
}
catch (DbConnectionException exp) {
    log.error("Failed to connect", exp);
}
```

### How to use logs to understand failures

Logs tell a story. The story of failure begins with a request or invocation of some piece of functionality, continues through a series of steps that may or may not help understand the failure, and finally concludes with the record of the error. Being able to see what happened before an error occurred is the first step to understanding why the error occurred.

## Telemetry

Telemetry is a reporting tool. It lets you see how well your code is doing what it’s supposed to do. Telemetry is different from logging in that it is intended to be used to create graphs that can show you how your code is doing overall, when looked at across all the users of the code.

The audience for telemetry is statistical methods. This means that telemetry should contain only things that a statistical
method can understand: numbers and categories. The kinds of measures that you might want to graph in this way include
such things as counts of success or failure or time spent doing various operations.

This snippet shows some kinds of telemetry that you might want to collect when querying a database.

```java
try {
    long startNanoseconds = System.nanoTime();
    metrics.addCount("db.query", 1);
    result = db.query(queryString, params);
    // if we reach here, the query did not throw an exception
    metrics.addCount("db.query.success", 1);
}
catch (DbQueryException exp) {
    metrics.addCount("db.query.failure", 1)
}
finally {
    long endNanoseconds = System.nanoTime();
    metrics.addLatency("db.query.latency", endNanoseconds - startNanoseconds);
}
```

## How to Use Telemetry and Logging

As a general rule, use telemetry to tell when there is something wrong, and use logging to figure out what the problem is.

Use telemetry to find out things like the percentage of queries that result in errors. In the above example, this would be
the total count of `db.query.failure` divided by the total count of `db.query`. We can also find out the average latency or p99 latency by collecting all of the `db.query.latency` entries and finding their average, or sort them and find the one that is
greater than 99% of the rest for p99 latency.

If we see that the error percentage (or "error rate") is higher than we like, we can use our logs to determine what is causing
the errors.
