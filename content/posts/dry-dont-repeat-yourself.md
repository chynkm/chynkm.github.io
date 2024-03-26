---
author: "Karthik M"
title: "DRY – Don’t repeat yourself"
date: "2018-09-16"
canonicalUrl: https://litebreeze.com/software-development/dry-dont-repeat-yourself/
tags:
- php
- dry
- software engineering
---

Excerpt from the book The Pragmatic Programmer:

_As programmers, we collect, organize, maintain, and harness knowledge. We document knowledge in specifications, we make
it come alive in running code, and we use it to provide the checks needed during testing._

_Unfortunately, knowledge isn’t stable. It changes often rapidly. Your understanding of a requirement may change
following a meeting with the client. The government changes a regulation and some business logic gets outdated. Tests
may show that the chosen algorithm won’t work. All this instability means that we spend a large part of our time in
maintenance mode, reorganizing and reexpressing the knowledge in our systems._

_Most people assume that maintenance begins when an application is released, that maintenance means fixing bugs and
nhancing features. We think these people are wrong. Programmers are constantly in maintenance mode. Our understanding
changes day by day. New requirements arrive as we’re designing or coding. Perhaps the environment changes. Whatever the
reason, maintenance is not a discrete activity, but a routine part of the entire development process._

_When we perform maintenance, we have to find and change the representations of things – those capsules of knowledge
embedded in the application. The problem is that it’s easy to duplicate knowledge in the specifications, processes, and
programs that we develop, and when we do so, we invite a maintenance nightmare – one that starts well before the
application ships._

_We feel that the only way to develop software reliably, and to make our developments easier to understand and maintain,
is to follow what we call the DRY principle:_

_Every piece of knowledge must have a single, unambiguous, authoritative representation within a system._

A concept is incomplete without a practical example. The following code is from one of my Laravel projects:

File: `TargetController.php`
```
public function guardActiveButton($target)
{
    if ($target->cio_venue_status == config('env.cio.venue_status.closed') || !$target->isBusinessHours()) {
        $activeButton = "closed_button";
    } elseif ($target->cio_mode == config('env.cio.mode.welcome_info')) {
        $activeButton = "welcome_button";
    } else {
        switch ($target->cio_traffic_light_status) {
            case config('env.cio.traffic_light_status.green'):
                $activeButton = "traffic_green_button";
                break;

            case config('env.cio.traffic_light_status.yellow'):
                $activeButton = "traffic_yellow_button";
                break;

            default:
                $activeButton = "traffic_max_button";
                break;
        }
    }

    return $activeButton;
}
```

File: `Target.php`
```
public function getIcon()
{
   if (
       $this->cio_venue_status == config('env.cio.venue_status.closed')
       || ! $this->isBusinessHours()
   ) {
        return 'closed.png';
   }

   if ($this->cio_mode == config('env.cio.mode.welcome_info')) {
       return 'welcome_info.png';
   }

   switch ($this->cio_traffic_light_status) {
       case config('env.cio.traffic_light_status.red'):
           return 'red.png';
           break;

        case config('env.cio.traffic_light_status.yellow'):
           return 'yellow.png';
            break;

        default:
           return 'green.png';
           break;
   }
}
```

File: `index.blade.php`
```
@php
if ($target->cio_venue_status == config('env.cio.venue_status.closed') || ! $target->isBusinessHours()) {
    $class = 'closed';
} elseif ($target->cio_mode == config('env.cio.mode.traffic')) {
    switch ($target->cio_traffic_light_status) {
       case config('env.cio.traffic_light_status.green'):
            $class = "open";
            break;

       case config('env.cio.traffic_light_status.yellow'):
            $class = "waiting";
            break;

       default:
            $class = "stop";
           break;
    }
} else {
    $class = 'welcome';
}
@endphp
```
If you analyse the above three excerpts, the conditions remain the same irrespective of the returned value. Redundant
code increases maintenance time. Moreover, if duplicated code requires modification, there’s a danger that one section
of the code is updated without checking further for other instances. This is especially problematic if changes are made
by a developer new to the project. We will adhere to the DRY principle by rewriting the above three code snippets into a
single function.

```
public function getDisplayInfo($group)
{
    $displayInfo = [
        'button' => [
            'closed' => 'closed_button',
            'welcome' => 'welcome_button',
            'red' => 'traffic_max_button',
            'yellow' => 'traffic_yellow_button',
            'green' => 'traffic_green_button',
        ],
        'css_class' => [
            'closed' => 'closed',
            'welcome' => 'welcome',
            'red' => 'stop',
            'yellow' => 'waiting',
            'green' => 'open',
        ],
        'icon' => [
            'closed' => 'closed.png',
            'welcome' => 'welcome_info.png',
              'red' => 'red.png',
            'yellow' => 'yellow.png',
            'green' => 'green.png',
        ],
    ];

    if (
        $this->cio_venue_status == config('env.cio.venue_status.closed')
        || ! $this->isBusinessHours()
    ) {
        return $displayInfo[$group]['closed'];
    }

    if ($this->cio_mode == config('env.cio.mode.welcome_info')) {
        return $displayInfo[$group]['welcome'];
    }

    switch ($this->cio_traffic_light_status) {
        case config('env.cio.traffic_light_status.red'):
            return $displayInfo[$group]['red'];
            break;

        case config('env.cio.traffic_light_status.yellow'):
            return $displayInfo[$group]['yellow'];
            break;

        default:
            return $displayInfo[$group]['green'];
            break;
    }
}
```

The three examples can now be replaced using the following method respectively:
```
$target->getDisplayInfo(‘button’);
$target->getDisplayInfo(‘icon’);
$target->getDisplayInfo(‘css_class’);
```

By following the DRY principle we have reused existing function and reduced knowledge duplication. DRY principle isn’t
limited to programming, it can be followed in any tasks. It is the one of the easiest fundamentals for a programmer to
learn, understand and implement in their daily tasks.
