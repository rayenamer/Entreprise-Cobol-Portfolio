
---

**Why Docker + TK5 is the right call**

- No Hercules configuration headaches — the container handles IPL, DASD images, device definitions, all of it
- Reproducible — blow it away, spin it back up, same clean state
- Your code lives on your host machine (Git, VS Code, normal tools), you just submit jobs into the container
- Works on any OS without touching your base system

---

**The architecture you want**

```
Your machine
├── VS Code + COBOL extension
├── src/
│   ├── payroll.cob
│   └── copybooks/
├── jcl/
│   └── payroll.jcl
└── scripts/
    └── submit.sh          ← talks to the container

Docker container (TK5)
├── Hercules (emulating mainframe hardware)
├── MVS 3.8j (real OS, real JES2 spool)
├── 3270 port exposed (for TSO/ISPF terminal)
└── card reader port exposed (for JCL submission)
```

---

**How job submission actually works**

MVS receives JCL the same way a real mainframe does — via a card reader device. Hercules exposes this as a port. Your submit script on the host does something like:

```bash
#!/bin/bash
# submit.sh — send a JCL job to MVS running in Docker
nc localhost 3505 < jcl/payroll.jcl
```

Port 3505 is the standard Hercules card reader port. `nc` (netcat) pipes your JCL file straight into the JES2 input queue. MVS picks it up, executes it, writes SYSOUT to the spool — exactly like a real mainframe job.

---

**What you need to get running**

1. Pull the TK5 Docker image (joergschultzelutter's Alpine-based one is a good starting point)
2. Map port `3270` (for your 3270 terminal — use x3270 or c3270)
3. Map port `3505` (card reader — for job submission)
4. Install the IBM COBOL or Broadcom COBOL extension in VS Code for syntax highlighting and copybook resolution
5. Write a small `submit.sh` wrapper so you can send jobs from your terminal without touching the container directly

---

**The workflow once it's set up**

1. Edit your COBOL in VS Code on your host
2. Write JCL that references your program
3. Run `./submit.sh` — job lands in JES2
4. Open your 3270 terminal, check the spool output in ISPF
5. Iterate

That's essentially the real mainframe developer loop, running entirely on your laptop for free. The only thing missing compared to IBM Z Xplore is the modern z/OS dialect differences — but the JCL, JES2, TSO/ISPF, and batch job mechanics are genuine.