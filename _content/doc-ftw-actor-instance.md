---
permalink: /docs/actor-instance/
layout:    docs
title:     Actor Instance
menu:      Actor Instance
authors:
  - Jack R. Dunaway
editors:
  - placeholder
---

An Actor Instance defines the actual procedural
abilities of an Actor, and does "actual work" to manipulate the real
or virtual world. Side effects created by an Actor Instance that could
be observed by another Actor Instance, such
as manipulating a physical instrument, or modifying a database table,
are usually always the sole "owners" of such

Sometimes "Actor Instance" is simply shortened as "Actor".

An Actor Instance may be a static dispatch call on a diagram, or a new
instance may be dynamically created via the `SpawnNewActor` method
of the Actor Connector class.

To launch multiple instances of an actor

{% include note.html type="protip" title="Protip: Actor Instance Reentrancy" body="When launching multiple instances of an Actor, ensure the Actor Process is reentrant. All instance methods should most likely be reentrant as well. As with typical reentrant design, static variables (such as uninitialized shift registers) and calls to non-reentrant blocking functions should be avoided." %}

An Actor Instance communicates with other Actor Instances
(or generally, other nodes capable of communicated with Actor
Instances) purely via messaging. Said another way, an Actor
never has static or dynamic dispatch callsites on another Actor's
methods

## Launching/Spawning a New Actor Instance

The `Featherweight-Actor-Instance` is designed such that the Actor Process
may be statically dispatched or dynamically dispatched.

### Static Launching of a Single Actor

The simplest way to launch an Actor is  Actor Process will

[screenshot of three actor instances]

## Actor Instance Configuration


### Quick Reference
<table class="ui celled table">
  <thead>
    <tr>
      <th>Configuration Element</th>
      <th>JSON Type</th>
      <th>Default Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code><b>Identity</b></code></td>
      <td>String (recommended)</td>
      <td><code>"FTW-Unnamed-Actor"</code></td>
    </tr>
    <tr>
      <td><code>InboxAddress</code></td>
      <td>String (required)</td>
      <td>-</td>
    </tr>
    <tr>
      <td><code>InboxConfiguration</code></td>
      <td><a href="../actor-inbox/">Inbox Object</a></td>
      <td><a href="../actor-inbox/">Default Inbox</a></td>
    </tr>
    <tr>
      <td><code>SupervisorAddress</code></td>
      <td>String (recommended)</td>
      <td><code>""</code></td>
    </tr>
    <tr>
      <td><code>DebugEnableInboxLogging</code></td>
      <td>Boolean</td>
      <td><code>false</code></td>
    </tr>
    <tr>
      <td><code>DebugEnableJobLogging</code></td>
      <td>Boolean</td>
      <td><code>false</code></td>
    </tr>
    <tr>
      <td><code>DebugShowPanel</code></td>
      <td>Boolean</td>
      <td><code>false</code></td>
    </tr>
    <tr>
      <td><code>HandshakeTimeout</code></td>
      <td>Integer (optional)</td>
      <td><code>500 (milliseconds)</code></td>
    </tr>
    <tr>
      <td><code>InitializationTimeout</code></td>
      <td>Integer (recommended)</td>
      <td><code>500 (milliseconds)</code></td>
    </tr>
    <tr>
      <td><code>LauncherAddress</code></td>
      <td>String (optional)</td>
      <td><code>"tcp://127.0.0.1:*"</code></td>
    </tr>
  </tbody>
</table>

### Configuration Details

#### Identity
Uniquely identifies an Actor Instance using a programmer-friendly naming scheme.
Used primarily for logging. FTW neither requires nor enforces uniqueness.

#### InboxAddress
Primary inbox address of the actor instance.

#### InboxConfiguration
Ruleset that governs how the actor reacts to Ask and Tell requests from remote
actors. Each inbox in the array includes address where it listens for requests,
its priority with respect to other inboxes, and message handling semantics for
different message types.

{% include note.html type="security" title="Default is permissive" body="The default values here are permissive, allowing all messages to be routed directly from the Inbox to the Instance." %}


#### SupervisorAddress


#### DebugEnableInboxLogging


#### DebugEnableJobLogging


#### DebugShowPanel
When `true`, the instance launched will show its front panel.

This debugging feature is not intended for to be used as an application facility
for showing user interfaces to users, but rather it is intended for developers
to interactively debug an Actor Instance.

#### HandshakeTimeout

#### InitializationTimeout
It is recommended to always explicitly set this value.

Choosing an optimal timeout value is not always simple, and that optimal value
will likely even change during the development cycle of the Actor Instance. Some
considerations when choosing this value:

* span of time required by Actor Instance to initialize, also accounting for
jitter and non-deterministic tasks (e.g., initializing hardware, connecting to
other actors, connecting to a database, ...)
* IDE with debugging versus RTE with debugging off
* load time if dynamically launching from disk
* target resource availability (especially CPU)


{% include note.html type="protip" title="Watch out for JIT compile time" body="When dynamically launching in the IDE, ensure that the Actor Instance and its static dependencies are already compiled for the current target. Opening a reference dynamically to source code that is not yet compiled for the current target will trigger a just-in-time compile which may add an unexpectedly long delay to the launch of the Actor. Unlike an intentional compile (triggered by opening the source in the IDE, or by mass compiling the project or source directory), this JIT compile is not cached by the environment, meaning that same penalty will continue to be incurred for subsequent launches." %}


#### LauncherAddress

{% include note.html type="notice" title="Note" body="The Featherweight Framework defines a few framework-level messages" %}


### Typical Example
{% highlight javascript %}
{
  "Identity": "FeatherweightActorExample",
  "InboxAddress": "tcp://127.0.0.1:*",
  "HandshakeTimeout": 1000,
  "DebugShowPanel": false
}
{% endhighlight %}

{% include note.html type="protip" title="Convenient placeholders" body="Configuration elements such as `DebugShowPanel` are often kept as part of the configuration, not because it is explicitly necessary to specify, but because it is convenient to toggle during development." %}

### Minimal Example
{% highlight javascript %}
{"InboxAddress":"ws://127.0.0.1:5555"}
{% endhighlight %}

Yep, that's it. The only configuration required to launch a FTW Actor Instance
is a valid, available InboxAddress.

### Full Example with InboxConfiguration
{% highlight javascript %}
{
  "Identity": "FeatherweightActorExample",
  "InboxAddress": "tcp://127.0.0.1:*",
  "InboxConfiguration": {
    "MaxMessageSize": 100000,
    "AdditionalServiceEndpoints": {
      "Primary": "tcp://127.0.0.1:*",
      "RemoteClients": "ws://*:*"
    },
    "MessageRoutingRules": {
      "FTW-PoisonPill": {
        "Accept": true,
        "Reject": false
      },
      "FTW-ActorStatus": {
        "Accept": true,
        "Reject": false,
        "BusyTimeout": 100,
        "ResponseIncludesIdentity": true,
        "ResponseIncludesTimestamp": true,
        "ResponseIncludesRelativeTimer": true
      },
      "Default": {
        "Accept": false,
        "Reject": false,
        "BusyTimeout": -1,
        "ResponseIncludesIdentity": true,
        "ResponseIncludesTimestamp": true,
        "ResponseIncludesRelativeTimer": true
      }
    }
  }
  "HandshakeTimeout": 1000,
  "DebugEnableInboxLogging": false,
  "DebugEnableJobLogging": false,
  "DebugShowPanel": false
}
{% endhighlight %}


## Framework Jobs

<table class="ui celled table">
  <thead>
    <tr>
      <th class="single line">Job</th>
      <th class="">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="single line"><code>FTW-Job-HandleIncomingRequest</code></td>
      <td>Placeholder</td>
    </tr>
    <tr>
      <td class="single line"><code>FTW-Job-HandleException</code></td>
      <td>Placeholder</td>
    </tr>
    <tr>
      <td class="single line"><code>FTW-Job-FailureToLaunch</code></td>
      <td>Ruleset that governs how the actor reacts to Ask and Tell requests from remote actors. Each inbox in the array includes address where it listens for requests, its priority with respect to other inboxes, and message handling semantics for different message types.</td>
    </tr>
    <tr>
      <td class="single line"><code>FTW-Job-Shutdown</code></td>
      <td>Placeholder</td>
    </tr>
    <tr>
      <td class="single line"><code>FTW: Exception Handler</code></td>
      <td>Placeholder</td>
    </tr>
  </tbody>
</table>

{% include note.html type="protip" title="Job section dividers in the case label" body="The job dividers (e.g., `--- Actor-Specific Jobs ---`) not only act as section dividers between different types of jobs in the FTW Actor Instance, but also act as convenient templates to duplicate as new jobs are created. This convenience is directly adapted from best-practice recommendations from the JKI State Machine." %}


## Centrality of Data

## Actors are Citizens
A good analogy of for an Actor Instance is to consider it a citizen
of a society (where, society is an analogy to `Actor Context`).

This society has limited resources (an Actor Context has finite RAM, CPU, etc...),
and for this reason the Actor Instance

## Centrality of Health
An Actor Instance is always expected to keep itself "healthy", sometimes
at the expense of fulfilling the requests of other Actors. This includes
per

## Assumed Statelessness between Actors

One important assumption to maintain is that all communication between
all Actors assumes no previous state of peer.

This means that even though a local endpoint may be aware that it is sending
messages in succession, it should not assume that the remote endpoint
is neither in an assumed state for subsequent messages nor even receiving
the messages in the same order as being sent.

For those of us who

## Cost of Failure vs Reasonable Expectations

The principles of Actor-Oriented design are not necessarily absolute rules.

For instance, it's often convenient to forget the "at most once" and
assume that messages sent will be reliably received.

The tradeoff and balance of this concession is based upon reasonable
expectations of reliability far exceed the cost of failure for infrequent
times when the expectation is not met.

A good example of a reasonable expectation of success and low cost of
failure is a touch-sensitive keypad on a microwave oven. While setting the time,
most of the times pushing the buttons works the first time, and in the event of failure,
the operator will simply press again, or perhaps clearing and correcting
an incorrect button. A less clear-cut scenario is using that same touchpad
to stop a microwave that is running as soup boils over. For this reason,
on most microwaves, there exists a much larger hardware button that stops operation
of the microwave that's easier to lunge toward (and more ergonomically suited
for lunging, with a contact area of several fingers or a fist instead of just
a fingertip!) Would it be cost-effective for the microwave manufacturer
to optimize success rate of the keypad from 99% to 99.99%? Perhaps not, given
the cost of failure is insignificant and easily-rectified. (Also, would the
"better" solution have been as easy to wipe off and clean?)

If codified, perhaps consider this heuristic: as expectation of success becomes
more and more probable, it's OK to tune the "paranoid" knob lower; as cost
of failure increases, tune the "paranoid" knob higher. "Paranoid", meaning additional
procedure and requisite syntax to develop and maintain as a remedy for
or reaction to foreseen failure modes.

### Design patterns and idioms for mitigating failure

#### Publishing level instead of level-change events
Some design patterns for mitigating failure include unconditionally publishing
state (level detection) versus expecting to catch a single message (level-change
detection). Since this could connote aliasing and missing state changes, when it's
critical to catch such short "pulses", a local detector on the publishing
endpoint would publish such occurences as a separate channel (e.g., in addition
to published "OverVoltage" as a boolean, "OverVoltageOccurrences" could be
a numeric counter. In the event that the OverVoltage condition was set and then
cleared between two publish periods, the receiver would receive two subsequent
messages stating that "OverVoltage" is false except that "OverVoltageOccurences"
has incremented by one.

{% include note.html type="protip" title="Remember" body="Continually publishing messages is one method of mitigating lost message, but even (and perhaps, especially these since bandwidth is increased) these messages may be lost." %}

## Tips on Dividing Applications into Actors

Let's create for ourselves a scenario for an application and consider
how we might divide application requirements into Actors Instances.

Consider this application requires:

* one voltage channel
* one current channel
* one output relay
* one algorithm and visualization for voltmeter data
* one algorithm and visualization for ammeter data
* one algorithm and visualization for voltmeter+ammeter data

One trivial manner of segmenting actors is to consider "one application;
one actor!"

Another trivial solution is "six things, six actors".

Or perhaps, we divide the hardware into one actor, put the the UI
into another actor, and defer the decision of where to do the calculations
since it seems easy enough to move

Are these options all valid? Yes! Are they optimal? Perhaps; perhaps no.

To begin to consider other compositions, we need to start considering
additional things. In the spirit of this example, let's tailor some
questions we might ask ourselves:

#### Are the three physical channels on the same bus?

If the three channels (2 input, 1 output) are on separate busses
(e.g., separate instruments with separate connections to that instrument),
likely they can reasonably be separated into two actor instances if desired.

If they are on the same bus, they very likely must exist within the same
actor instance, because even though the channels may be physically independent,
the bus acts as a shared resource between the independent channels. If we were to
place control into separate actors, we might encounter collisions over this
shared resource. For example, which actor initializes/deinitializes the device?
What happens physically when the bus receives two simultaneous commands - does
the protocol return an error for one, or queue one after the other? Regardless
the specific workaround for such hard/specific/discretionary questions, it might
have been easier to avoid them altogether.

In this manner,
we might visualize the instrument bus as an actor system itself.

The "Actor Connector" to this system is the hardware bus connection which either
continually publishes measurement data, or has a Request/Reply semantic
implementing "Get Voltage" and "Get Current", or perhaps "Get Measurement" that
returns a structure of both measurements.

{ include note.html type="notice" title="Cool tip (need sunglasses glyph)" text="Instruments with remote control protocols map exceptionally well to Actor Systems. They typically maintain their own health (e.g., a 10W power supply might return an error if commanded to 1 megawatt), define a published interface" }

#### Do the two physical channels have an implicit shared resource - time?

If the measurements must be timed with respect to the other, this timing
requirement acts as an oft-overlooked

## Public vs. Private vs. Published Interface

The interface of an Actor is a bit like and a bit unlike that of a
traditional object-oriented class whose methods dispatch on the call stack
of the caller.

The traditional OO API has:

* public methods/properties/fields: any code may dispatch and use
* protected methods/properties/fields: only descendent classes may dispatch;
  non-descendents that attempt to use will throw a compiler error about scope
* private methods/properties/fields: only methods of the class itself can
  link to or dispatch these; even descendent classes will throw an error

An Actor, on the other hand, has:

* No "public" methods/properties/fields
* Protected methods/properties/fields: may be used by the actor instance,
  or children of the actor instance
* Published requests ("methods"): the list of "methods", roughly analogous
  to remote procedure calls (RPC), that the actor will respond to. The "published"
  interface

### Published Interface

This is the list of messages

{% include note.html type="notice" title="Note" body="The Featherweight Framework defines a few framework-level messages" %}

### Published Data
An actor is able to broadcast arbitrary data to remote endpoint(s) who
choose to connect to the Actor's publisher endpoint(s).

Each actor instance chooses a rate at which it desires to publish. This could be
periodic (e.g., every 100msec), or sporadic (e.g., notification of an event).

Each actor may define [0, N) publishers on which it publishes arbitrary
data to each channel.

## Inheritance of Actor Instances

The Actor Reference Design in Featherweight allows inheritance. The simplest
example is how application specific Actors you write will likely inherit from
`FTW-Actor.lvclass`.

{% include note.html type="warning" title="Heads up!" body="It's possible to create your own reference designs that compose and wrap core FTW-ActorInstance functions, but this is not necessarily recommended." %}
