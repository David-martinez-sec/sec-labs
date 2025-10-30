# Metasploit Cheat Sheet — Lab-Safe

> **Important:** This document is written for _authorized learning in isolated labs_ (TryHackMe, HackTheBox, your own VMs). **Do not** use these techniques or commands against systems you do not own or do not have explicit written permission to test.

---

## Quick reference — msfconsole basics

These are common console commands you will use during lab work. Replace placeholder values (e.g., `RHOSTS`, `LHOST`) with lab-specific addresses.

```text
# Launch msfconsole
msfconsole

# Search for modules (exploit/auxiliary/post/payload)
search <keyword>
# Example: search smb

# Select a module to use
use <module/path>
# Example: use exploit/windows/smb/ms17_010_eternalblue

# Show information about the loaded module
info

# Show configurable options for the selected module
show options

# Set module options
set <OPTION> <value>
# Example: set RHOSTS 10.10.1.5

# Unset a value
unset <OPTION>

# Show payload options (if a payload is selected or required)
show payloads
show options

# Run the module
run
# or
exploit

# If a module has an interactive prompt that returns you to msfconsole
back

# Background an interactive session (meterpreter) and return to msfconsole
background

# List active sessions
sessions -l

# Interact with a session
sessions -i <session-id>

# Kill a session
sessions -k <session-id>

# Exit msfconsole
exit
```

---

## Database / host discovery (using msfconsole's DB)

If Metasploit's database is enabled, you can import and query scan results. Use `db_nmap` to run nmap and import results into the Metasploit DB.

```text
# Check DB status
db_status

# Run nmap and import results into the database
db_nmap -sV -p- 10.10.1.0/24

# List hosts discovered in DB
hosts

# List services discovered
services

# Show credentials harvested/stored
creds

# Show loot (files grabbed by post modules)
loot

# Search for vulnerabilities recorded in DB
vulns
```

> Tip: `db_nmap` is convenient in labs because the results are available to the rest of the framework (e.g., `hosts`, `services`).

---

## Typical high-level workflow (lab-focused)

1. **Start msfconsole** — `msfconsole`
2. **Recon / enumeration** — use `db_nmap` or external `nmap` to map live hosts and services.
3. **Search for modules** — `search <service|version|cve|keyword>` to find relevant auxiliary or exploit modules.
4. **Inspect module** — `use <module>`, then `info` and `show options` to see required settings and targets.
5. **Set required options** — `set RHOSTS`, `set RPORT`, `set THREADS`, `set SMBUser`, etc. Always review `show options` after setting.
6. **Select payload (if applicable)** — `show payloads` then `set PAYLOAD <payload/name>` and `show options` for the payload.
7. **Run safely** — `run` or `exploit` in a lab VM or intentionally vulnerable target.
8. **Post-test** — list `sessions -l`, `sessions -i <id>` to interact. Use post modules with care and only in labs.
9. **Cleanup** — `sessions -k`, remove artifacts, revert snapshots.

---

## Useful msfconsole commands (compact list)

- `search <term>` — Find modules by keyword.
- `use <module>` — Load a module to the current context.
- `info` — Show detailed info about the loaded module.
- `show options` — Show options the module expects.
- `show payloads` — List payloads compatible with the current module.
- `set <OPTION> <value>` — Configure options (module or payload).
- `unset <OPTION>` — Remove a previously set option.
- `run` or `exploit` — Execute the module.
- `back` — Unload the current module and return to the root prompt.
- `sessions` — Manage active sessions. Common subcommands: `-l` (list), `-i <id>` (interact), `-k <id>` (kill).
- `background` — Background the current interactive session.
- `db_nmap` — Run nmap with results stored in msf DB (same syntax as nmap).
- `hosts`, `services`, `creds`, `loot`, `vulns` — Query the database.
- `spool / spool off` — Capture console output to a file (useful for documenting results).

---

## Example lab-safe command sequence (with placeholders)

```text
# 1) Start msfconsole
msfconsole

# 2) Perform a safe, permitted scan and import results
db_nmap -sV -p22,80,139,445 10.10.1.5

# 3) View hosts and services discovered
hosts
services

# 4) Search for modules related to a service found (example: smb)
search smb

# 5) Pick a module conceptually (e.g., an SMB exploit module) and inspect
use <module/path>
info
show options

# 6) Set module options using placeholders
set RHOSTS 10.10.1.5
set RPORT 445
set THREADS 10

# 7) Show options again and choose payload if required
show options
show payloads
set PAYLOAD <payload/name>
set LHOST 10.10.1.100

# 8) Run in the lab
run
# or
exploit

# 9) Manage sessions
sessions -l
sessions -i 1
background
sessions -k 1

# 10) Cleanup and document
spool /path/to/output.log
exit
```

---

## Post-module notes (lab context)

- Always `sessions -l` to identify active sessions created by modules.
- Use `background` to drop a session into the background so you can run other modules.
- Use `post/*` modules only within your lab to collect evidence or demonstrate concepts (e.g., `post/multi/gather/` modules). Do not run them on unauthorized systems.

---

## Safety, ethics, and documentation checklist

- Confirm scope and written authorization before testing.
- Use VM snapshots and revert after testing.
- Take notes: `spool` console output, capture screenshots, and save `pcap` files if network captures are relevant.
- Clearly label all documentation as lab-only and include timestamps and revert actions.

---

## Quick references & learning resources

- Official Metasploit documentation and `msfconsole --help`.
- TryHackMe / HackTheBox labs for safe practice.
- Rapid7 blog and Metasploit module documentation for learning module-specific behaviors.

---

*End of document.*

