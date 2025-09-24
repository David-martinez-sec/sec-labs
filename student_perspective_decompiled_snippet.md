# Learning Exercise: Analyzing a Sanitized Decompiled Snippet for Threat Hunting

> **Note:** This is an *educational exercise* for students learning about **threat hunting** and **threat intelligence**.  
> It contains **no runnable malware code** — everything is conceptual, sanitized, and safe.  
> Think of it as practice material for building an analyst’s mindset.

---

## 1. Decompiled (sanitized) snippet — learning example
Below is a **sanitized pseudo-code snippet** that mimics what an analyst might see when reverse engineering malware.  
As a student, your job is to recognize what each part means and how it connects to threat hunting.

```c
// ---------- Entry point ----------
int main()
{
    // Step 1: load configuration data
    config = load_config_from_resource_or_file("<CONFIG_BLOB>");

    // Step 2: ensure persistence
    if (!check_autorun_exists("<AUTORUN_MARKER>")) {
        write_file("%APPDATA%\<LAUNCHER>.exe", stub_bytes);
        register_autorun("<AUTORUN_MARKER>", "%APPDATA%\<LAUNCHER>.exe");
    }

    // Step 3: create a mutex (so only one copy runs)
    mutex_handle = create_mutex("Global\<MUTEX_NAME>");

    // Step 4: decrypt hidden command-and-control (C2) endpoints
    c2_list = decrypt_config(config.encrypted_endpoints, config.key_hint);

    // Step 5: main beaconing loop
    while (true) {
        sleep(randomize_interval(config.beacon_interval));
        beacon = gather_system_fingerprint();
        resp = send_http_post(c2_list.choose(), "/check", beacon);
        if (resp && resp.commands) {
            foreach (cmd in resp.commands) {
                dispatch_command(cmd);   // jump into command handler
            }
        }
    }
}
```
---

## 2. Student Learning Focus: What does each part mean?

- **Configuration blob**  
  - Malware often hides its server list or settings in encrypted blobs.  
  - *Threat intel skill:* recognize when files contain high-entropy (random-looking) data.  
  - *Threat hunting tip:* search for executables with large encrypted resource sections.

- **Persistence marker**  
  - The code tries to stay alive by adding itself to autorun keys.  
  - *Threat intel skill:* persistence artifacts (Run keys, scheduled tasks) become IoCs.  
  - *Threat hunting tip:* query for newly created autorun entries in EDR/registry logs.

- **Mutex creation**  
  - Prevents multiple copies from running.  
  - *Threat intel skill:* mutex names are often reused by families → useful for attribution.  
  - *Threat hunting tip:* hunt for unusual named mutexes in endpoint telemetry.

- **Beaconing**  
  - Regular "phone home" requests to a server.  
  - *Threat intel skill:* note the jitter (random sleep), used to evade detection.  
  - *Threat hunting tip:* look for periodic outbound traffic at odd intervals to rare domains.

- **Command dispatcher**  
  - The malware waits for instructions (download, exfiltrate, self-remove).  
  - *Threat intel skill:* recognize modular designs (loaders vs plugins).  
  - *Threat hunting tip:* correlate downloads immediately followed by process creation or DLL loading.

---

## 3. Example Threat Intelligence Artifacts (safe placeholders)

As a student, practice listing artifacts you would share with your team:

- Persistence marker: `<AUTORUN_MARKER>`  
- Mutex name: `Global\<MUTEX_NAME>`  
- File paths: `%APPDATA%\<LAUNCHER>.exe`, `%APPDATA%\plugins\`  
- Behavior: periodic beaconing, plugin downloads, exfiltration uploads  

👉 **Exercise:** How would you translate these into IoCs (hashes, YARA, Splunk queries)?

---

## 4. Threat Hunting Practice Scenarios

Try these student exercises:

1. **EDR Hunt:** Find processes writing executables to `%APPDATA%` and then adding registry Run keys.  
2. **Network Hunt:** Look for hosts making small, periodic POST requests with jittered timing.  
3. **Memory Analysis:** Imagine running this in a sandbox — what decrypted values (like C2 endpoints) would you expect to see in memory?  
4. **Threat Intel Sharing:** How would you share the mutex name and autorun marker with peers safely?

---

## 5. How this connects to Threat Hunting & Threat Intel

- **Threat Hunting:** Proactive searching for abnormal behaviors (autorun creation, beaconing traffic, mutex usage).  
- **Threat Intelligence:** Turning observed details (mutexes, file paths, persistence methods) into reusable knowledge that others can detect and defend against.

---

## 6. Final Notes for Students

- This exercise is **about patterns and thinking like an analyst**.  
- You don’t need real malware samples to practice; use pseudo-code and reports.  
- Focus on connecting **observed behaviors → IoCs → hunting queries → defensive actions**.  
- Always use safe environments and curated labs when moving beyond theory.

---

**Next step idea:** Map each line of the pseudo-code to a **MITRE ATT&CK tactic** (like Persistence, Command & Control, Exfiltration). This will help you build a structured mental model of attack behaviors.

