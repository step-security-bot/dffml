# 2023-03-21 Engineering Logs

- Today we see alignment inbound across supply chain security and the interplanetary virtual machine
  - We seek to bridge ideation to production via CI/CD pull request validation flows into deployment in a hermetic (cacheable) execution environment such as IPVM. This requires alignment across provenance formats and invocation auth formats. Essentially, if there was a valid CI/CD build, deploy it. (It doesn't necessarily require it, but it will make security much more clean from an auditability and trackability perspective and if it can all go back to JSON-LD then query is easy, which means we can feed it back into Alice's training loop and she can hypothesize and execute experiments lickity split). It also means application of policy becomes uniform across ideation and production environments, hopefully reducing policy escapes, aka lack of alignment to strategic plans and principles. This is how we get our feedback from the behavioral analysis portion of the Entity Analysis Trinity
    - https://github.com/ipvm-wg/spec/pull/8
- https://openatintel.podbean.com/e/threat-modeling-down-the-rabbit-hole/
  > I'm wondering if there's anything, if there's any angle here that we haven't covered that you wanted to make sure to mention. Speaking of, you know, different tooling that you can use, right, we have this project where we're looking at, you know, defining, when you look at the threat model of an application, you're also looking at, you know, the architecture, right, you know, what are the components in that. And so one of the things that John and I realized when we went about, you know, the tooling saga in our threat model journey is that there's a lot of different tools, right, and there's always going to be a different tool that's better for something else, right. So we began to focus on kind of this approach of more like, well, you know, what are the key components, right? And then how do we, you know, expose those to the tools and source from the tools as appropriate, right, or, you know, as context appropriate, right? So we, so we've come up with this concept, right, of this, we basically said, we want to describe the architecture. We would like to do this in an open source way. So we took the word open and we took the word architecture and we put them together and now we've got the open architecture. And so the goal here is really to say, okay, well, what does the application look like? And to speak of the angles, we've got this Entity Analysis Trinity, which basically says, you know, what, what are you trying to do? What is your threat model, right? And then what are you actually doing? And what did you write down, right? What is your code? So what is your intent at the top of the triangle, right? What is your static analysis say? And what is your sort of behavioral or dynamic analysis say, right? And so the objective here overall is to, you know, apply your static analysis methodologies, apply your dynamic analysis, right? You know, maybe that's telemetry from the field or whatever, right, to tell you about, you know, what's happening in your software, or, you know, what does it look like when it's tested under a live, you know, dynamic scanning environment, right? And how does that relate to your threat model, right? And so we can do that because we can identify the different components being tested by the different tools and map them into this, you know, open description of architecture

![EATv0.0.2](https://user-images.githubusercontent.com/5950433/188203911-3586e1af-a1f6-434a-8a9a-a1795d7a7ca3.svg)