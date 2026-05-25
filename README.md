# Active Directory Lab

I am a second-year Computer Information Systems student at Cal Poly Pomona working toward breaking into the IT industry. I am mainly looking for a helpdesk or IT support role. One of the best ways I found to build real skills is to actually get my hands dirty and gain hands on experience, so I built this lab from scratch.

This repository documents my personal Active Directory home lab. Everything here was built, broken, fixed, and documented by me. The goal was simple, stop just reading about Active Directory and actually use it the way IT professionals do every day. 

Throughout this lab I practiced the core tasks a helpdesk technician handles daily — creating and managing user accounts, organizing departments through Organizational Units, controlling access through Security Groups, enforcing policies through Group Policy Objects, and supporting users through Remote Desktop. Every step is documented with the reasoning behind it, not just the clicks.

---

## Environment

| Component | Details |
|---|---|
| **Host Hypervisor** | Oracle VirtualBox |
| **Domain Controller** | Windows Server 2022 — DC01 |
| **Client Machine** | Windows 10 Pro — CLIENT01 |
| **Domain** | homelab.local |
| **DC IP Address** | 192.168.10.1 |
| **Client IP Address** | 192.168.10.2 |
| **Network** | Dual Adapter — NAT (internet) + Internal Network (HomeLab-Network) |
