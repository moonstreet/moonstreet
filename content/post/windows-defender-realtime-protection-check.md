---
title: "Windows Defender Realtime Protection Check"
date: 2019-01-04T12:11:56+01:00
draft: false
image: "uploads/settings.jpeg"
tags: ["cybersecurity"]
---

# Windows Defender 


is tegenwoordig een prima anti-virus programma, mits real-time protection werkt.
Check of real-time protection werkt en niet wordt geblocked door een firewall via onderstaand commando:
 
```
C:\Program Files\Windows Defender>mpcmdrun -validatemapsconnection
```

p.s. werkt alleen in een command prompt met <b>admin privileges</b>.
