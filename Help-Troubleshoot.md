# ❓ Qemu-Vortax V 2.0 - Help and Troubleshooting

This guide provides solutions to the most common issues encountered when running the Qemu-Vortax workflow.

---

## 🚫 Connection Issues

### 1. The VNC / RDP / NoMachine connection fails or times out.

| Cause | Solution |
| :--- | :--- |
| **Incorrect Auth Keys** | Ensure that both `TAILSCALE_AUTHKEY` and `NGROK_AUTH` secrets are correctly added to your repository settings. |
| **Tailscale Network Issue** | If using Tailscale, ensure your local device (the one you are connecting from) is also running and logged into Tailscale. |
| **Ngrok Tunnel Not Starting** | Check the action logs to ensure the Ngrok tunnel started successfully and provided a public URL (`RDP_URL`, `VNC_URL`, etc.). |
| **VM Not Started** | The VM might take 3-5 minutes to fully boot. Wait for the log message "VM started successfully." |

### 2. I can connect, but the screen is black or I only see the default desktop.

| Cause | Solution |
| :--- | :--- |
| **QEMU VM Not Displayed** | You must launch the QEMU display session. **If connecting via RDP or NoMachine, double-click the `Connect_VNC.sh` script on the remote desktop to open the QEMU window.** |

### 3. VNC connection asks for a password and then fails (New V2.0 issue).

| Cause | Solution |
| :--- | :--- |
| **Incorrect VNC Password** | Ensure the password you enter into the VNC viewer matches the exact value you provided in the `vnc_password` input when running the workflow. |

### 4. Bliss OS QCOW2: Screen is black after VNC login.

| Cause | Solution |
| :--- | :--- |
| **Bliss OS Sleep Mode (QCOW2)** | **Bliss OS QCOW2** (disk image version) often enters a deep standby mode after booting, resulting in a black screen. This typically occurs with static images (QCOW2) and not Live Boot (ISO). |
| | **To fix:** You need to force a wakeup or soft restart from the QEMU Monitor. |
| | 1. Press **Ctrl + Alt + 2** on your keyboard to switch to the QEMU Monitor console. |
| | 2. Type the command: `system_reset` |
| | 3. Press **Ctrl + Alt + 1** to return to the main VNC screen after the reboot is complete. |

---

## 💻 Workflow & Boot Issues

### 5. Bliss OS ISO: Codes appear on the screen and the operation halts.

| Cause | Solution |
| :--- | :--- |
| **Bliss OS ISO Compatibility** | The **Bliss OS ISO** (Live Boot) version may encounter compatibility issues or hang when codes appear during the boot phase due to graphics rendering requirements. |
| | **Solution:** It is **highly recommended** to use the **`qcow2`** option for Bliss OS instead of **`iso`** for a stable and guaranteed run. Ensure you set `boot_mode` to `qcow2`. |

### 6. The workflow fails immediately during the "Determine source URL" step.

| Cause | Solution |
| :--- | :--- |
| **Mismatch between Choice and Boot Mode** | You selected a `system_choice` (e.g., `kali`) but chose the wrong `boot_mode` (e.g., selected `iso` when only a `qcow2` link is available for that system). **Fix:** Change `boot_mode` to `qcow2` or choose a system that supports `iso`. |
| **Invalid Custom URL** | The URL provided in `source_url` is broken or inaccessible. **Fix:** Verify the link works in a browser. |

### 7. The workflow gets stuck during the "Prepare files and disks" step.

| Cause | Solution |
| :--- | :--- |
| **Large File Download** | If the file is large (several GBs) and hosted on MediaFire/Mega, the `badown` tool can take a long time. Check the logs for `Downloading... X%` progress messages. |
| **Archive Extraction Failure** | The archive file might be corrupted or not supported by 7z/unzip. |

### 8. The VM boots to the QEMU console, not the graphical desktop.

| Cause | Solution |
| :--- | :--- |
| **Missing Desktop Environment** | This happens if you used a pre-configured cloud image (`centos_qcow2`, `debian_qcow2`) which usually boots to the command line. **Fix:** These images are typically command-line only. If you need a GUI, use an ISO or a desktop-edition QCOW2 image. |

---

## 💾 Data Persistence (MEGA Upload V2.0)

### 9. QCOW2 upload failed even though it was requested.

| Cause | Solution |
| :--- | :--- |
| **Insufficient Time or File Size** | Upload failure is usually due to one of two reasons: **A. Insufficient Time:** There was not enough time left to complete the file **compression and upload** to MEGA. **B. File Size Limit Exceeded:** The size of the final QCOW2 file (after compression) exceeded the size limit of the generated free account, currently **20 GB**. |
| | **Solution:** To ensure successful upload, allow extra `runtime` beyond your actual usage needs. Avoid using very large files that may exceed 20 GB. |

### 10. The QCOW2 file was not uploaded to MEGA.

| Cause | Solution |
| :--- | :--- |
| **Upload Skipped** | The input `upload_qcow2` was set to `no`. **Fix:** Set the input to `yes` for your next run. |
| **QEMU Did Not Stop** | The process might have failed to kill QEMU after runtime. Check the log for "Session ended after X minutes" followed by compression attempts. |

### 11. How do I access my uploaded file?

| Cause | Solution |
| :--- | :--- |
| **Accessing File** | Look for the **"Data Persistence (MEGA Upload)"** section at the very end of the run logs. It contains the following critical information: |
| | 1. **MEGA Account Credentials** (Email and Password). |
| | 2. **Download Link** for the compressed QCOW2 file. |

---

## 🔧 Advanced Troubleshooting

### 12. The VNC screen is distorted, glitchy, or the resolution is too high/low.

| Cause | Solution |
| :--- | :--- |
| **Display Resolution Mismatch** | The default resolution set by the VM might be incompatible with the VNC or RDP client. **Fix:** Try changing the display resolution *inside* the VM's desktop environment (XFCE Settings). If the issue persists, the kernel video driver might be missing or faulty. |

### 13. My mouse or keyboard input is laggy, jumpy, or unusable.

| Cause | Solution |
| :--- | :--- |
| **Network Latency** | The primary cause is the delay between your device and the GitHub Actions runner. **Fix:** Switch your `connection_method` from `ngrok` to **`tailscale`** (if possible), as Tailscale generally offers lower latency and a more stable connection. Also, ensure your local network connection is stable. |

### 14. I want to use my previously uploaded QCOW2 file in a new run. How?

| Cause | Solution |
| :--- | :--- |
| **Reusing Uploaded Image** | You need to use the image link (and potentially the MEGA credentials) from your previous run. **Fix:** |
| | 1. Find the **Download Link** of your compressed QCOW2 file in the previous run's logs (under "Data Persistence"). |
| | 2. Paste this link into the **`source_url`** input for your new run. |
| | 3. Set the **`boot_mode`** to **`qcow2`**. |

### 15. Why did the VM shut down before the `runtime` officially ended?

| Cause | Solution |
| :--- | :--- |
| **GitHub Actions Time Limit** | The QEMU process might have been killed by the GitHub Actions platform itself, not your `runtime` input. **GitHub Actions runners have a hard limit of 6 hours (360 minutes) for the entire job.** |
| | **Fix:** Ensure your specified `runtime` is significantly less than 360 minutes (e.g., max 340-345 minutes) to allow time for setup, downloading, and the final MEGA upload/cleanup steps. |

---

## 💡 General Tips

* **Log Visibility:** The V2.0 update minimizes initial logs. Expand the steps like "Install XFCE + RDP + NoMachine" if you need detailed diagnostic information.
* **F8 Disconnect:** To cleanly disconnect from a VNC session when running inside an RDP/NoMachine window, press **F8** on your client keyboard, and select "Disconnect".
