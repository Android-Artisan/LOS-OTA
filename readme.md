# 📡 Unofficial LineageOS OTA Updates via GitHub Releases

This guide explains how to set up **OTA (Over-the-Air) updates** for an **unofficial LineageOS build** using **GitHub Releases** and **GitHub Actions**. Your ROM users will automatically receive updates without manual intervention.

---

## 📌 Features

- ✅ Uses **GitHub Releases** to host ROM builds  
- ✅ No need for an external OTA server  
- ✅ Automatically updates \`ota.json\` when a new ROM is uploaded  
- ✅ Fully compatible with **LineageOS built-in Updater**  

---

## 🚀 Step 1: Modify \`build.prop\` to Enable OTA Updates

To allow your ROM to check for OTA updates, modify the **\`build.prop\`** file in your **LineageOS source**.

1. Go to:  
   \`\`\`
   vendor/your_device/build.prop
   \`\`\`

2. Add or modify this line:
   \`\`\`ini
   ro.ota.update_url=https://raw.githubusercontent.com/YOUR_GITHUB_USERNAME/YOUR_REPO/main/ota.json
   \`\`\`

3. **Save the file** and **rebuild your ROM**.

👉 Replace **\`YOUR_GITHUB_USERNAME/YOUR_REPO\`** with your actual **GitHub username and repository name**.

---

## 🚀 Step 2: Create \`ota.json\` in Your GitHub Repo

Your ROM needs an **\`ota.json\`** file to check for new updates.

1. Go to your **GitHub repository**.  
2. Click **Add file > Create new file**.  
3. Name the file: **\`ota.json\`**  
4. Paste this content:
   \`\`\`json
   {
     "response": [
       {
         "datetime": 1710000000,
         "filename": "lineage-20.0-UNOFFICIAL-yourdevice.zip",
         "id": "abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890",
         "size": 1200000000,
         "url": "https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO/releases/latest/download/lineage-20.0-UNOFFICIAL-yourdevice.zip",
         "version": "20.0"
       }
     ]
   }
   \`\`\`

5. Click **Commit new file** to save.

### ℹ️ Explanation:
| Field      | Description                       |
|------------|----------------------------------|
| \`datetime\` | UNIX timestamp of the build      |
| \`filename\` | Name of the ROM ZIP file        |
| \`id\`       | SHA256 checksum (optional)      |
| \`size\`     | File size in bytes              |
| \`url\`      | Download URL from GitHub        |
| \`version\`  | LineageOS version               |

---

## 🚀 Step 3: Automate OTA Updates Using GitHub Actions

Set up **GitHub Actions** to automatically update \`ota.json\` whenever you upload a new ROM.

1. Go to your **GitHub repo**.  
2. Navigate to \`.github/workflows/\`.  
3. Click **Add file > Create new file**.  
4. Name the file: **\`ota-updater.yml\`**  
5. Paste this workflow code:
   \`\`\`yaml
   name: OTA Updater

   on:
     release:
       types: [published]

   jobs:
     update-ota:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout Repository
           uses: actions/checkout@v4

         - name: Get Latest Release Details
           id: get_release
           run: |
             LATEST_RELEASE=\$(curl -s https://api.github.com/repos/\${{ github.repository }}/releases/latest)
             echo "FILENAME=\$(echo \$LATEST_RELEASE | jq -r '.assets[0].name')" >> \$GITHUB_ENV
             echo "FILESIZE=\$(echo \$LATEST_RELEASE | jq -r '.assets[0].size')" >> \$GITHUB_ENV
             echo "FILEURL=\$(echo \$LATEST_RELEASE | jq -r '.assets[0].browser_download_url')" >> \$GITHUB_ENV
             echo "DATETIME=\$(date +%s)" >> \$GITHUB_ENV
             echo "ID=\$(sha256sum <(echo \$LATEST_RELEASE) | cut -d ' ' -f1)" >> \$GITHUB_ENV

         - name: Update OTA JSON
           run: |
             echo '{
               "response": [
                 {
                   "datetime": '\${{ env.DATETIME }}',
                   "filename": "'\${{ env.FILENAME }}'",
                   "id": "'\${{ env.ID }}'",
                   "size": '\${{ env.FILESIZE }}',
                   "url": "'\${{ env.FILEURL }}'",
                   "version": "20.0"
                 }
               ]
             }' > ota.json

         - name: Commit and Push OTA JSON
           run: |
             git config --global user.name "github-actions"
             git config --global user.email "actions@github.com"
             git add ota.json
             git commit -m "Update OTA JSON"
             git push origin main
   \`\`\`

6. Click **Commit new file**.

---

## 🚀 Step 4: Upload a ROM to GitHub Releases

Every time you upload a new ROM, your **\`ota.json\`** will update automatically.

### How to Upload a ROM:
1. Go to your **GitHub repo**.  
2. Navigate to **Releases > Draft a new release**.  
3. Add your version tag (e.g., \`v20.0\`).  
4. Upload your ROM ZIP file.  
5. Click **Publish release**.

---

## 🚀 Step 5: Test OTA Updates

1. Flash your ROM on your device.  
2. Go to **Settings > System > Updater**.  
3. Click **Check for updates**.  
4. If an update is available, it will **download and install automatically**.

---

## 🎯 Bonus: Schedule Weekly Builds

You can schedule automatic OTA builds every week by adding this to your GitHub Actions file:
\`\`\`yaml
on:
  schedule:
    - cron: "0 0 * * 0"  # Every Sunday at midnight
\`\`\`

---

## 🎉 Congratulations! OTA Updates Are Fully Automated!

Now your **Unofficial LineageOS Build** will support **OTA updates automatically** via **GitHub Releases**.

---

### 🔑 Notes
- All data will be stored in **GitHub Releases**  
- Your users will always get the **latest ROM**  
- No external servers needed  

---

## 📄 License

This guide is open-source and free to use. Feel free to modify it for your custom ROM project.

---

## 💬 Support & Contact

If you need help, **open an issue** or reach out to ROM development communities.  
Happy building! 🚀
EOF
