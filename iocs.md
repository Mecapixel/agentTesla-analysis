# Indicators of Compromise — AgentTesla Loader (tmzZ.exe)
**Pixora Inc. | May 9, 2026**

---

| Type | Value | Notes |
|---|---|---|
| SHA-256 | 09783fa659755aba97256704aaf284df35d698ee9aff8cecd0eeeabe3a576a89 | Sample file hash |
| Campaign Tag | IowsSzd | Decoded from AT string |
| Hardcoded Key | RE-808 | reelEpoch parameter |
| Payload Size | 97,280 bytes | Extracted from bitmap |
| Embedded Resource | Resources.Tint (100,072 bytes) | Steganographic carrier image |
| Hex String 1 | 496F7773 | Decodes to: Iows |
| Hex String 2 | 737A64 | Decodes to: Szd |
| Base64 String | U3lzdGVtLkFjdGl2YXRvcg== | Decodes to: System.Activator |
| Entry Point | DoodleJumpMini.Program.Main | Fake game entry |
| Loader Method | FormMenu.ArchiveSpoolDigest() | Steganographic extraction method |
| Build Timestamp | 5/6/2026 10:48:22 AM | Compilation timestamp |
