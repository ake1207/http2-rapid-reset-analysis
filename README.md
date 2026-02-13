# HTTP/2 Rapid Reset DDoS Attack – Technical Analysis

TL;DR: The 2023 HTTP/2 Rapid Reset attack demonstrated how fully protocol-compliant behaviour could be weaponised to generate nearly 200 million requests per second while keeping attacker cost relatively low. By rapidly creating and cancelling HTTP/2 streams, attackers exploited multiplexing features designed for performance, creating a severe asymmetry between attacker effort and defender resource consumption at global scale.

For the full original reflective write-up, see:  
[Full Analysis](./full-analysis.md)

## Overview

I decided to take a deeper look into the HTTP/2 Rapid Reset DDoS attack not only because it was a record-breaking traffic spike that Cloudflare mitigated, but because of the creativity and subtlety of the attack itself. Unlike traditional volumetric DDoS attacks that rely on sheer overwhelming bandwidth, this attack abused standard protocol behaviour in a way that made malicious traffic difficult to distinguish from legitimate usage. Cloudflare mitigated traffic reaching nearly 200 million requests per second — roughly three times larger than the previous record. What stood out most was that the traffic consisted of valid HTTP/2 frames, not malformed packets.

## Background

HTTP/2 allows clients to open multiple independent streams over a single TCP connection. This feature enables:
  -Efficient request handling
  -Reduced connection overhead
  -Improved performance for modern web applications

This efficiency is critical at Cloudflare’s scale, where the company supports major platforms such as Discord, Reddit, Cisco, and even government infrastructure.
However, the Rapid Reset attack exploited this exact feature.

## How the attack worked

The attack relied on repeatedly:
  •Opening HTTP/2 streams
  •Sending requests
  •Immediately cancelling (resetting) those streams
  •repeating hundreds of millions of times per second

Because the HTTP/2 protocol supports rapid stream creation and cancellation under valid protocol rules, attackers could induce significant server-side processing without:
  -Maintaing long-lived connections
  -Transferring large amounts of data 
  -Sending malformed traffic

By rapidly sending and resetting streams, attackers maximised server-side work rather than maximising bandwidth usage.
Notably the attack was carried out using a botnet of roughly 20,000 machines - relatively small by modern standards, highlighting the efficiency of the attack.

## Why was it effective

This attack was sophisticated because it shifted the objective, Instead of maximising bytes per second, it maximised computational work per request.
Some of the Key characteristics are:
  •Protocol-compliant behaviour
  •Minimal bandwidth usage
  •High server processing cost
  •Evasion of traditional DDoS detection signals

The attackers optimised for low cost to themselves and high cost to defenders, creating a strong asymmetry.

## Defensive challenges at worldwide scale

Defending against this attack was particularly challenging because:
  •Cloudflare handles legitimate HTTP/2 traffic from millions of clients.
  •Browsers frequently reset streams for valid reasons such as:
    •Users scrolling quickly
    •Cancelling image loads
    •Closing tabs
  •Blocking too aggressively risks disrupting real users.
  •Blocking too slowly allows system degradation.

This highlights a core difficulty at global scale, Distinguishing malicious behaviour from legitimate high-performance protocol usage without harming availability.
Cloudflare’s existing protections absorbed most of the attack, initially affecting around 1% of requests. Working alongside Google and AWS, Cloudflare implemented targeted mitigations to detect abusive stream reset patterns while preserving valid HTTP/2 behaviour.

## Key takeaways

•The attack exploited features designed for performance and efficiency. Protocol design assumptions can become liabilities under extreme adversarial pressure.
•The attackers created disproportionate defensive burden with relatively low resources: highlighting the attackers were aiming for a cost asymmetry.
•The traffic was valid, the impact on the server arose not from malicious packets, but from behaviour that stressed and poked at protocol limits.
•At a global scale, distinguishing genuine from malicious behaviour is really complex.

## Personal reflection

This incident changed my outlook of DDoS attacks. Prior to this, I associated DDoS primarily with bandwidth saturation. The Rapid Reset attack emancipated that protocol-level standards can be leveraged to create large-scale disruptions without the traditional approach of overwhelming a server with raw traffic volume. 
Raising further questions:
  •What other protocol-level assumptions could be exploited?
  •How can defensive systems anticipate future abuse before it manifests?
  •Could multiplexed systems like HTTP/3 or SSH introduce similar asymmetries?

The most impactful lesson from this for me is that security isn't static, it's ever changing. As systems evolve for performance and efficiency, new liabilities may emerge. The challenge, and what I find most fascinating about security, is anticipating those liabilities before attackers exploit them.

## References

https://blog.cloudflare.com/technical-breakdown-http2-rapid-reset-ddos-attack/
