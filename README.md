## Objective

- Configure IP addressing on all router interfaces and end devices
- Enable RIP (version 2) on every router
- Advertise all directly connected networks
- Verify full end-to-end connectivity between all LANs

## Topology
<paste your topology here, e.g.>

PC0 ---- SW0 ---- R1 ====== R2 ====== R3 ---- SW1 ---- PC1
                    (serial / ethernet links)
| Device | Interface | IP Address | Subnet Mask | Connected To |
|--------|-----------|------------|-------------|--------------|
| R1 | `<Gig0/0>` | `<192.168.1.1>` | `<255.255.255.0>` | LAN 1 |
| R1 | `<Se0/0/0>` | `<10.0.0.1>` | `<255.255.255.252>` | R2 |
| R2 | `<Se0/0/0>` | `<10.0.0.2>` | `<255.255.255.252>` | R1 |
| R2 | `<Se0/0/1>` | `<10.0.0.5>` | `<255.255.255.252>` | R3 |
| R3 | `<Gig0/0>` | `<192.168.2.1>` | `<255.255.255.0>` | LAN 2 |

| Device | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|-------------|-----------------|
| PC0 | `<192.168.1.10>` | `<255.255.255.0>` | `<192.168.1.1>` |
| PC1 | `<192.168.2.10>` | `<255.255.255.0>` | `<192.168.2.1>` |

## Configuration

### 1. Interface addressing (example: R1)

```cisco
enable
configure terminal
hostname R1

interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit

interface Serial0/0/0
 ip address 10.0.0.1 255.255.255.252
 clock rate 64000
 no shutdown
 exit
```

> `clock rate` sirf DCE side par lagta hai.

### 2. RIP configuration

```cisco
router rip
 version 2
 no auto-summary
 network 192.168.1.0
 network 10.0.0.0
 exit

end
write memory
```

Same tareeqe se har router par uske **directly connected** networks advertise karein.

- `version 2` — classless routing, VLSM support, subnet mask updates ke saath bhejta hai
- `no auto-summary` — subnets ko classful boundary par summarize hone se rokta hai

---

## Verification

| Command | Kya check karta hai |
|---------|---------------------|
| `show ip route` | Routing table — RIP routes `R` se start hote hain |
| `show ip protocols` | RIP running hai, version, advertised networks |
| `show ip interface brief` | Interfaces up/up hain ya nahi |
| `show running-config` | Poori configuration |
| `debug ip rip` | Live RIP updates (band karne ke liye `undebug all`) |

**Connectivity test:**

```
PC0 > ping 192.168.2.10
PC0 > tracert 192.168.2.10
```

Expected: saare pings successful, aur `show ip route` par har router ko remote networks `R` codes ke saath nazar aayen.

---

## Key Concepts

- **RIP** ek distance-vector protocol hai jo **hop count** ko metric ke taur par use karta hai
- Maximum **15 hops** — 16 ka matlab network unreachable hai
- Har **30 seconds** par routing updates broadcast/multicast hote hain
- RIPv2 multicast address **224.0.0.9** use karta hai (RIPv1 broadcast karta tha)
- Administrative distance: **120**

---

## How to Run

1. Cisco Packet Tracer install karein (version `<8.x>` ya us se upar)
2. Repository clone ya download karein
3. `Rip_routing.pkt` open karein
4. Kisi bhi PC par Desktop → Command Prompt se dusre LAN ke PC ko ping karein

---

## Troubleshooting

| Problem | Possible Fix |
|---------|-------------|
| Ping fail ho raha hai | PC par default gateway set hai? |
| Interface down | `no shutdown` laga hua hai? |
| Serial link down | DCE side par `clock rate` set karein |
| Remote network route nahi aa raha | `network` statement chhoot gaya hai — `show ip protocols` check karein |
| Subnets galat merge ho rahe hain | `no auto-summary` add karein aur `version 2` confirm karein |

---

## Author

`<Your Name>` — `<GitHub profile link>`
