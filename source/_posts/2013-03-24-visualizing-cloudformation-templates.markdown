---
layout: post
title: "Visualizing CloudFormation templates"
date: 2013-03-24 19:45
comments: true
categories:
---

I've been using [CloudFormation] [cloudformation] a lot recently to
manage AWS resources. I'm impressed with a lot of the functionality it
provides and it has allowed us to delete a lot of code on my current
project which was needed to handle edge cases in provisioning AWS
resources (eventual consistency issues, for example).

After a couple of months using it seriously, I only have four
complaints about CloudFormation:

 * Stack lifecycle operations (create/update and delete) are not
   idempotent.
 * Notification topics can only be set up at stack-creation time, so
   if the topic gets accidentally deleted there is no way to get
   notifications from the stack.
 * Creation and modification of resources in the stack is serialized,
   which makes operations on large stacks very slow.
 * The template syntax is _horrible_: hard to write and even harder to
   read.

<!--more-->

I can't claim to have a better solution to the syntax problem. Given
the constraints, the current syntax is probably reasonable; and the
language itself is extremely well thought-out. Perhaps the long-term
solution is to treat the JSON as an intermediary form and write
bindings for individual languages to generate it.

However, as things stand, once stacks have more than a couple of
resources in them, I find it hard to keep track of what is going on.
So I've written [a tool] [cfviz] to vizualize templates. It reads
template JSON and spits out [Graphviz] [graphviz] dot format which can
be used to generate an image.

{% img /images/EC2_Untargeted_Launch_with_EBS_Volume.svg 400 'EC2 untargeted launch with EBS volume' %}

The examples in this post are from the [sample templates]
[aws-samples] provided by AWS. You can find generated images for all
those samples [here] [sample-images].

{% img /images/AutoScalingMultiAZWithNotifications.svg 700 'Auto-scaling multi-AZ with notifications' %}

I'd love to hear from you if you think this is useful.

[aws-samples]: http://aws.amazon.com/cloudformation/aws-cloudformation-templates/
[cfviz]: https://github.com/benbc/cloud-formation-viz
[cloudformation]: http://aws.amazon.com/cloudformation/
[graphviz]: http://www.graphviz.org/
[sample-images]: https://github.com/benbc/cloud-formation-viz/tree/master/samples
