---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: architectures.png
# apply any unocss classes to the current slide
class: 'text-center'
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
transition: slide-left
title: Welcome to Slidev
mdc: true
---

# Ports and Adapters

For People on Tight Schedules 

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" flex="~ justify-center items-center gap-2" hover="bg-white bg-opacity-10">
    Press Space for next page <div class="i-carbon:arrow-right inline-block"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <div class="i-carbon:edit" />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>


---
transition: fade-out
---

# Welcome!

Agenda of this talk:

- 📝 **Hi there!** - who's talking to you
- 🎨 **Ports and Adapters 101** - a quick reminder / TL;DR for those new to it
- 🧑‍💻 **The app: High-level context** - the problem, top-level components
- 🤹 **The app: serverless architecture** - the implementation
- 🎨 **The tight schedules challenges** - the flipside of having fancy stuff
- 🎨 **The realization** - let's cut short the waiting using the P&A architecture!
- 🧑‍💻 **The effect** - development speed boost
- 🛠 **Small gotchas** - the things we needed to sort out along the way
- 🧑‍💻 **Questions** - anything you'd fancy to ask

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

<!--
Hi! I'm Przemek, a long-time Scala enthusiast, and a happy developer. At work, I'm maintaining a content management system
for an Audio on Demand media company. My friend, Tomek Godzik (of Metals fame), has noted, that I wear clothes from various Polish software shops,
not having worked for any one (yet! I love what they are doing, and I am rooting for all of them).

Today I want to share my first-hand experience of applying the Ports&Adapters architecture to get a nice development speed boost.

During my talk, I first want to refresh the basic information about that architecture.

Then, I want to give you a context of the problem I needed to solve, going from the high to the low level.

We'll see that fancy stuff can come with some serious disadvantages, especially when you need to meet some corporate standards.

I'll show you how I noticed the potential benefits of applying Ports&Adapters, how I implemented it, and what were the benefits.

Finally, I'll show you some gotchas I had along the way.
-->

---
transition: fade-out
layout: image
image: architectures.png
backgroundSize: contain
---
# P&A
# 101

<!-- The Hexagonal architecture, Ports and Adapters, The Clean Architecture, The Onion Architecture
- all of them (and their multiple variants) share one important trait.

Please tell me, can you see the arrows going from the application services to the databases? (who can see it, please raise your hand)

Exactly, you can not! That's the point. All the arrows go towards the center, which is the application logic.

(these architectures also share a less obvious, but important trait, which is reliance on interfaces; the particular components
can be swapped for their alternatives, as long as the component interface matches)
-->

---
transition: fade-out
layout: image
image: layers-vs-not.png
backgroundSize: contain
---
# P&A
# in a
# Small
# app

<!-- Many people will be familiar with the conventional, layered, approach.
If we consider a database-powered REST API microservice, here's how it would look like.

Below, the same microservice implemented in the Ports and Adapters way.
Please note that the last arrow changes its direction and nature. Instead of being a simple call,
it becomes an inheritance/subtyping (extends) relation. Some types from the Database package extend types
from the Application package.

We'll see this in detail on the following slide.
-->
---
transition: fade-out
layout: image
image: p-a-a-zoom-in.png
backgroundSize: contain
---
# Ports and Adapters in a Small app: zoom in

<!-- Please note, since the Application module is independent, we can use it with a different
database adapter, WITHOUT having to carry the old database module around.

This is a crucial difference.

(In the layered systems, the common policy is that the code of the layer N should call the layer N+1,
yet that's not enforced by the design, so often you see violations, like usage of the database types in the HTTP code;
OTOH, if the arrows go towards the center, the violations are impossible - not to say there is no way around that,
but it's more effort, and it requires a conscious decision to move some database types
to the application module)
-->
---
transition: fade-out
layout: image
image: player.jpeg
backgroundSize: contain
---
# High
# level
# use
# case

<!-- At work, we had the following problem:
we need to allow a team of content curators to edit the configuration of the audio content in an 
Audio on Demand player application.

Among other things, they need to be able to select the content, pick the layouts,
and provide configuration for algorithms.
-->
---
transition: fade-out
layout: image
image: high-level.png
backgroundSize: contain
---
# High
# level
# blocks

<!-- The company chose a third party CMS for the Content Curation team.
The Player application wanted to receive the configuration through custom domain events.

Our application were to receive a webhook, use the CMS API to fetch the context information
required to produce the events, then to produce the domain events.
-->
---
transition: fade-out
layout: image
image: serverless-arch.png
backgroundSize: 70% 70%
---
# Our serverless vision

<!-- 
We have split the processing into stages for better reliability, mostly against two problems:
- webhook timeouts (SQS)
- CMS API rate limits (the lambda gets retried, and can have a dead-letter-queue)

development env:
- a monorepo with SBT subprojects
- AWS CDK for IaC
- CI/CD on GHE
-->
---
layout: image-right
image: pablo.png
backgroundSize: contain
---
# The tight schedules challenge

- deployments take time
- the business stakeholders do not (always) care, everyone wants their features shipped ASAP
- unit tests not always cut it
- what if you need to profile the app?
- what if you need to take a memory dump?

<!-- 
- an exploratory testing digression
	- when NO - certainly don't go for them if it's easy to write good tests
	- when YES
		- when you want to catch the unit testing blind spots
		- when writing of the suitable tests is not practical (the number of things to be stubbed/mocked or 65MB JSON test examples)
		- when some kind of problems surface only for sufficiently large inputs
		- when you want to investigate a complicated case, occurring in certain environment only
- the issue of environment reservation: "who works in the test environment?"
- the (code line changed to executed in an environment) cycle time:
	- clean compilation, tests, packaging, docker image security audit at CI [ pablo1 ]
	- upload of the Docker image to the Elastic Container Registry [ pablo2 ]
	- CloudFormation stacks' deployment (with AWS CDK) to all supported AWS regions [ pablo3 ]
	- CMS UI interactions to set up the test data and trigger the webhooks
	- checking the DataDog logs
- everyone accepts the time overhead as something obvious, a sad fact of life
-->
---
transition: fade-out
---
# Realization

<v-clicks depth="1">

- The interesting logic in each lambda can be isolated
- What is the simplest possible alternative for required input/output adapters?
- Let's replace the inputs (SQS, Kinesis) with standard input
- Let's replace the outputs (DDB, Kinesis) with standard output
- `fetch-context cms_entry::123234345 | jq | tee context.json | pbcopy`
- `cat context.json | print-events | jq | pbcopy`

</v-clicks>

---
transition: fade-out
---
# The new reality
<v-clicks depth="1">

- feature iteration time
    - down from 20 minutes to 30 seconds
- profiling capabilities
- memory analysis

</v-clicks>
<!--
Imagine that soon after the refactoring of the Domain Events Lambda we ran into an increased memory consumption problem,
occurring for some large payloads. We scheduled a task for investigating it, which (in our expectations) needed to easily
take 2 weeks of work.

The ability to run the lambda logic locally saved us over a week of work.

Then, maybe two weeks after this, we encountered a performance problem happening for CMS items with large dependency trees
in the Context Fetcher Lambda. The lambda was timing out. Luckily, we were already preparing the command-line version
of that lambda, so the experimentation with an improved context fetching approach could be made much more effectively.

Finally, maybe a month later, we've run into another performance problem, this time it was the Domain Events Lambda
timing out. We were able to easily run the async profiler to quickly spot the problematic parts of the processing.
-->

---
layout: image-right
image: ensure-is-up-to-date.png
---

# The SBT gotcha

SBT universal packager caveat

<v-clicks depth="1">

- you package (`sbt stage`) the app
- you edit the source code
- you run the app
- whoopsie!

</v-clicks>

<!--
If you are used to (spoiled by?) scala-cli, you take it for granted that the program never gets out of date.
SBT native packager script cannot assume anything so it simply executes what is built at the moment.

Hence, the gotcha.

Now, should you run (the slow-starting) SBT on every program invocation? That would be a disaster.

Our solution was to use a poor-man/good-enough change detection in bash, to trigger the build only
when necessary.
-->

---
---

# The logging gotcha

The serverless processes like to log
<v-clicks depth="1">

- you capture the output
- you want to process it
- whoopsie! `jq: parse error: Expected string key before ':' at line 1, column 3`
</v-clicks>
<v-clicks depth="1">
```scala
import cats.effect.IO

object StandardOut {

  type Println = Any => IO[Unit]

  def setupStandardOutput: IO[Println] = IO {
    val originalStandardOutputStream = System.out
    System.setOut(System.err)
    (any: Any) => IO(originalStandardOutputStream.println(any))
  }

}
```
</v-clicks>

<!--
The standard output is a great place to print the program results to (as it enables piping it to other programs
or redirecting it to files).

Yet a problem appears when some libraries also want to also use it for their own purposes.

At our company we've got a logging library which uses the standard output. It was too much hassle at the time
to add the required configurability to it, so we resorted to a trick.

We capture the original standard output at the very beginning of the program execution and save it for the code responsible
for printing the results. Then, we order the JVM to plug the standard error stream in place of the standard output.

That way, all logs end up in the standard error stream, which makes them visible in the terminal (a useful thing!) but
they don't mix with the primary result, which can be piped, transformed, and parsed freely.
-->

---
transition: fade-out
---
# Questions?

---
transition: fade-out
---
# Image attribution

- The onion architecture from https://dev.to/jnavez/make-your-microservices-tastier-by-cooking-them-with-a-sweet-onion-34n2
- The clean architecture from https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
- The hexagonal architecture from https://alistair.cockburn.us/hexagonal-architecture
- P&A from https://www.dossier-andreas.net/software_architecture/ports_and_adapters.html



