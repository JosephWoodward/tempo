---
title: "Tail-based sampling"
---

# Tail-based sampling

Tempo aims to provide an inexpensive solution that makes 100% sampling possible.
However, sometimes constraints will make a lower sampling percentage necessary or desirable,
such as runtime or egress traffic related costs.
Probabilistic sampling strategies are easy to implement,
but also run the risk of discarding relevant data that you'll later want.

In tail-based sampling, sampling decisions are made at the end of the workflow.
The Grafana Agent groups spans by trace ID and makes a sampling decision based on the data contained in the trace.
For instance, inspecting if a trace contains an error.

To group spans by trace ID, the Agent buffers spans for a configurable amount of time,
after which it will consider the trace complete.
Longer running traces will be split into more than one.
However, waiting longer times will increase the memory overhead of buffering.

One particular challenge of grouping trace data is for multi-instance Agent deployments,
where spans that belong to the same trace can arrive to different Agents.
To solve that, in the Agent is possible to distribute traces across agent instances by consistently exporting spans belonging to the same trace to the same agent.

This is achieved by redistributing spans by trace ID once they arrive from the application.
The Agent must be able to discover and connect to other Agent instances where spans for the same trace can arrive.
For kubernetes users, that can be done with a [headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services).

Redistributing spans by trace ID means that spans are sent and receive twice,
which can cause a significant increase in CPU usage.
This overhead will increase with the number of Agent instances that share the same traces.

<p align="center"><img src="../tail-based-sampling.png" alt="Tail-based sampling overview"></p>

## Quickstart

To start using tail-based sampling, define a sampling policy.
If you're using a multi-instance deployment of the agent,
add load balancing and specify the resolving mechanism to find other Agents in the setup.
To see all the available config options, refer to the [configuration reference](https://github.com/grafana/agent/blob/main/docs/configuration-reference.md#tempo_instance_config).

```
tempo:
  configs:
    - name: default
      ...
      tail_sampling:
        policies:
          - rate_limiting:
              spans_per_second: 50
        load_balancing:
          resolver:
            dns:
              hostname: host.namespace.svc.cluster.local
```
