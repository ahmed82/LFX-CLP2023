# Notes on the BDLS Paper : Byzantine Fault Tolerance in Partial Synchronous Networks
Paper : https://eprint.iacr.org/2019/1460.pdf, will refer to this as the BDLS Paper in the write up.

# A Brief of Synchrony, Asynchrony and Partial Synchrony
Reference : https://decentralizedthoughts.github.io/2019-06-01-2019-5-31-models/  

In the standard distributed computing model, the communication uncertainty is captured by an adversary that can control the message delays. The three communication models are:
1. Synchronous model : For any message sent, the adversary can delay its delivery by at most Δ (a known finite time bound )
2. Asynchronous model : For any message sent, the adversary can delay its delivery by any finite amount of time. There's no bound on the time to deliver a message but, but each message must eventually be delivered.
3. Partial synchrony : Aims to find middle ground between these two models. The assumption is that there exists some known finite time bound Δ and a special event called GST (Global Stabilization Time) such that:
    * The adversary must cause the GST event to eventually happen after some unknown finite time.
    * Any message sent at time x must be delivered by time Δ+max(x,GST).
    * Informally, the system behaves asynchronously till GST and synchronously after GST.

## Drawbacks of Synchronous Models: 
* Setting a large and conservative Δ of say 10 minutes may indeed always faithfully model the real world. However, protocol designers who depend on Δ may incur very long timeouts and hence degrade performance.
* Setting a small and aggressive Δ of say 0.1 seconds may actually not always faithfully model the real world. This means that protocols whose safety depends on this bound may suffer safety violations in the real world.
* Even if we found a magical sweet spot for Δ, imagine a sender that broadcasts a message to two receivers, one message arrives after Δ−ϵ time and the other after Δ+ϵ. Here the real world would behave differently than the model and this could again potentially cause safety problems.

## Asynchronous Model and its Drawbacks
* Asynchronous model forces protocol designers to assume nothing about network delays.
* Since they do not depend on any time bound, message delays cannot cause unexpected safety violations.
* Since they cannot use any fixed values for timeouts, they must inherently adapt to the actual latency of the system.  
