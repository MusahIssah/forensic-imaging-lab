# 🔍 Forensic Imaging Lab — Creating a Forensic Image for Investigation

**Environment:** Windows Server 2022 (Hyper-V VM — MUS-DC-01)  
**Tool:** OSForensics v10 by PassMark Software  
**Date:** June 2026  
**Author:** Musah Issah | [LinkedIn](https://linkedin.com/in/musah-issah-925ba2313) | [GitHub](https://github.com/MusahIssah)

---

## 📌 Overview

Digital forensics is a critical component of incident response. When a security incident occurs, investigators must create a **forensic image** — a bit-for-bit copy of a storage volume — before any analysis begins. This ensures the original evidence is preserved and untouched.

This lab simulates a real-world forensic acquisition workflow: preparing source and storage partitions, installing a forensic tool, creating a case, acquiring a raw disk image with cryptographic hash verification, and mounting the image as a read-only volume for examination.

---

## 🎯 Learning Objectives

- Prepare a **Data Source** partition and a dedicated **Image Storage** partition
- Install **OSForensics** and create a structured investigation case
- Acquire a forensic disk image using **Direct Sector Copy** method
- Verify image integrity using **SHA-256** (primary) and **MD5** (secondary) hashes
- Mount the forensic image as a **read-only** volume using PassMark OSFMount
- Confirm the mounted image is forensically sound (no write access)

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| Windows Server 2022 | Lab environment (Hyper-V VM) |
| Disk Management | Partition preparation |
| OSForensics v10 (PassMark) | Case management & forensic imaging |
| PassMark OSFMount | Mounting forensic images |
| SHA-256 / MD5 | Cryptographic hash verification |

---

## 🗂️ Lab Architecture

```
Disk 0 (99.37 GB)
├── C:\ — OS Volume (74.96 GB NTFS) — Boot/Page File
├── Image Storage (I:) — 14.65 GB NTFS  ← forensic image saved here
└── Data Source (F:) — 9.76 GB NTFS     ← source volume to be imaged

Disk 1 (59.98 GB)
└── DATA (E:) — 59.98 GB NTFS
```

---

## 🔬 Walkthrough

### Task 1 — Prepare the Data Source & Image Storage Partitions

Before imaging, two dedicated partitions were created: one to serve as the **evidence source** (Data Source F:) and one to store the forensic image output (Image Storage I:).

**Step 1:** In Disk Management, right-click the C: volume and select **Shrink Volume**. Enter `10000 MB` to carve out unallocated space.

![Shrink C Drive](screenshots/01-disk-management-shrink-c-drive.png)

---

**Step 2:** Use the **New Simple Volume Wizard** to format the new partition as NTFS. Label it `Image Storage`.

![Format Image Storage Partition](screenshots/02-new-volume-format-image-storage-partition.png)

---

**Step 3:** Confirm the final disk layout. Three key volumes are now present: C:, Data Source (F:), and Image Storage (I:).

![Disk Management Overview](screenshots/03-disk-management-all-partitions-overview.png)

---

**Step 4:** Populate the Data Source (F:) partition by copying files from the C: drive — this simulates the evidence data that would exist on a suspect machine.

![Copying Files to Data Source](screenshots/04-copying-files-to-data-source-partition.png)

---

### Task 2 — Install OSForensics & Create a New Case

**Step 5:** Install OSForensics (by PassMark Software). Once complete, launch the application.

![OSForensics Setup Complete](screenshots/05-osforensics-setup-wizard-complete.png)

---

**Step 6:** Review the OSForensics dashboard. Note the workflow panel covering Case Management, File Searching, Hashing, and Viewers — this is a full digital forensics platform.

![OSForensics Dashboard](screenshots/06-osforensics-main-dashboard.png)

---

**Step 7:** Navigate to **Manage Case → New Case**. Browse to Image Storage (I:) and create a new folder named `images` for the case output.

![Browse for Case Folder](screenshots/07-osforensics-manage-case-browse-folder.png)

---

**Step 8:** Fill in the Basic Case Data. Set the Case Name to `Data Source Investigation`, Acquisition Type to **Investigate Disk(s) from Another Machine**, and point the Case Folder to a custom location.

![New Case Basic Data](screenshots/08-osforensics-new-case-basic-data.png)

---

**Step 9:** Confirm the case folder is set to `I:\images\`. Enable **Log case activity** for a full audit trail.

![Case Custom Location Set](screenshots/09-osforensics-new-case-custom-location.png)

---

**Step 10:** The case is successfully created. "Data Source Investigation" now appears in the case list with creation timestamp (Monday, June 8, 2026, 20:02:57).

![Case Created Successfully](screenshots/10-osforensics-case-created-data-source-investigation.png)

---

### Task 3 — Create the Forensic Image & Mount for Analysis

**Step 11:** Navigate to **Create Forensic Image → Create Disk Image**. In the Save dialog, navigate to Image Storage (I:) and name the file `Data Source Image`. Save as **RAW Image (.img)**.

![Save Image Dialog](screenshots/11-osforensics-forensic-imaging-save-image-dialog.png)

---

**Step 12:** Configure **Image Settings**:
- **Source Disk:** `\\.\PhysicalDrive0: Partition 4, F: [9.76GB NTFS]`
- **Target:** `I:\Data Source Image.img`
- **Primary Hash:** SHA2-256
- **Secondary Hash:** MD5
- ✅ Verify Image File After Completion
- ✅ Disable Shadow Copy

![Image Settings Configuration](screenshots/12-osforensics-image-settings-hash-configuration.png)

> ⚠️ **Key Forensic Principle:** Hash verification ensures the forensic copy is mathematically identical to the source. If the hashes match post-acquisition, the image is forensically sound and admissible.

---

**Step 13:** Imaging begins. Live statistics confirm:
- Copy Method: **Direct Sector Copy**
- Speed: **230.9 MB/s**
- SHA-256 and MD5 hashes computed in real time as sectors are read

![Imaging In Progress](screenshots/13-osforensics-imaging-in-progress-with-hashes.png)

---

**Step 14:** Imaging completes successfully.
- **Status:** Imaging Successfully Completed
- **Data Read:** 9.76 GB = Disk Size 9.76 GB (100%)
- **Unreadable Data:** None
- **Primary Hash (SHA-256):** `cfa10a826b02b1cc793b271070db07fbaaf8fbc7168d66206075361b2589cf09`
- **Secondary Hash (MD5):** `16b018f502cd9e4ad681b3833063c52d`

![Imaging Complete](screenshots/14-osforensics-imaging-successfully-completed.png)

---

**Step 15:** Use **PassMark OSFMount** to mount the image. Navigate to Image Storage (I:), select `Data Source Image.img`, and confirm the `.img.info.txt` metadata file was also generated.

![OSFMount Select Image](screenshots/15-osforensics-osfmount-select-image-file.png)

---

**Step 16:** The mounted image appears as **Data Source (G:)** in Windows Explorer — a new read-only virtual drive. The `Users` folder is visible, confirming a successful mount.

![Mounted Image Drive](screenshots/16-mounted-image-data-source-g-drive.png)

---

**Step 17:** Browsing the mounted image (G:) reveals the Users folder with three accounts: `Administrator`, `OktaService`, and `Public` — all timestamped from the acquisition. Right-clicking confirms **no "New Folder" option** — the volume is read-only and cannot be modified.

![Mounted Image Read-Only](screenshots/17-mounted-image-users-folder-read-only-view.png)

---

**Step 18:** Side-by-side comparison — the original C:\ Users folder (Local Disk) shows the same three accounts but with the real last-modified timestamps, confirming the forensic image is an accurate, unmodified copy of the source.

![Source vs Image Comparison](screenshots/18-original-source-users-folder-comparison.png)

---

## ✅ Results & Verification

| Check | Result |
|-------|--------|
| Image format | RAW (.img) |
| Copy method | Direct Sector Copy |
| Total data read | 9.76 GB |
| Disk size | 9.76 GB |
| Unreadable data | None |
| SHA-256 (primary) | `cfa10a826b02b1cc793b271070db07fbaaf8fbc7168d66206075361b2589cf09` |
| MD5 (secondary) | `16b018f502cd9e4ad681b3833063c52d` |
| Image verified post-acquisition | ✅ Yes |
| Mounted as read-only | ✅ Confirmed — no write access |
| Case activity logged | ✅ Yes |

---

## 🔐 Forensic Principles Demonstrated

**Chain of Custody** — Case created in OSForensics with logging enabled before acquisition began, establishing a timestamped audit trail.

**Evidence Integrity** — SHA-256 and MD5 hashes generated during acquisition and verified post-copy. Matching hashes prove the image is a forensically sound, unaltered copy of the source partition.

**Write Blocking** — The forensic image, once mounted via OSFMount, is read-only. This mirrors the function of a hardware write blocker — preventing any modification to the evidence copy during analysis.

**Non-Destructive Analysis** — All examination is performed on the image, not the original source partition, preserving the evidence in its original state.

---

## 📁 Repository Structure

```
forensic-imaging-lab/
├── README.md
├── docs/
│   └── methodology.md
└── screenshots/
    ├── 01-disk-management-shrink-c-drive.png
    ├── 02-new-volume-format-image-storage-partition.png
    ├── 03-disk-management-all-partitions-overview.png
    ├── 04-copying-files-to-data-source-partition.png
    ├── 05-osforensics-setup-wizard-complete.png
    ├── 06-osforensics-main-dashboard.png
    ├── 07-osforensics-manage-case-browse-folder.png
    ├── 08-osforensics-new-case-basic-data.png
    ├── 09-osforensics-new-case-custom-location.png
    ├── 10-osforensics-case-created-data-source-investigation.png
    ├── 11-osforensics-forensic-imaging-save-image-dialog.png
    ├── 12-osforensics-image-settings-hash-configuration.png
    ├── 13-osforensics-imaging-in-progress-with-hashes.png
    ├── 14-osforensics-imaging-successfully-completed.png
    ├── 15-osforensics-osfmount-select-image-file.png
    ├── 16-mounted-image-data-source-g-drive.png
    ├── 17-mounted-image-users-folder-read-only-view.png
    └── 18-original-source-users-folder-comparison.png
```

---

## 🔗 Related Projects

- [Okta AD Integration Lab](https://github.com/MusahIssah) — Active Directory to Okta provisioning workflow
- [Okta Threat Detection Lab](https://github.com/MusahIssah) — Credential stuffing & impossible travel detection with Python
- [JML Automation Pipeline](https://github.com/MusahIssah) — Joiner-Mover-Leaver automation with Make.com & Okta API

---

## 👤 About

**Musah Issah** — IAM Analyst & SOC Analyst | Okta Certified Professional | CompTIA Security+  
U.S. Air Force Reservist | DoD Secret Clearance  
📍 Bronx, NY  
🔗 [linkedin.com/in/musah-issah-925ba2313](https://linkedin.com/in/musah-issah-925ba2313)
