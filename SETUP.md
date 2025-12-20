# Scoop Bucket Setup Instructions

This directory contains all the files needed for the `scoop-arcitems` repository.

## Step 1: Create the Repository

1. Go to GitHub and create a new repository: `scoop-arcitems`
2. Make it public
3. Don't initialize with README (we have our own)

## Step 2: Push Files to Repository

```bash
cd scoop-bucket-files
git init
git add .
git commit -m "Initial Scoop bucket setup"
git branch -M main
git remote add origin https://github.com/pdavlin/scoop-arcitems.git
git push -u origin main
```

## Step 3: Configure Repository Settings

1. Go to Settings > Actions > General
2. Under "Actions permissions", select "Allow all actions and reusable workflows"
3. Under "Workflow permissions", select "Read and write permissions"
4. Click "Save"

## Step 4: Update the Manifest with Current Version

Before testing, update `bucket/arcitems.json`:

1. Get the latest release version from https://github.com/pdavlin/arcitems/releases
2. Get the SHA256 hash for `arcitems-windows-amd64.zip` from the release's `checksums.txt`
3. Update the `version`, `url`, and `hash` fields in the manifest

Example:
```bash
# Get latest release info
VERSION=$(curl -s https://api.github.com/repos/pdavlin/arcitems/releases/latest | jq -r .tag_name | sed 's/^v//')
CHECKSUM=$(curl -s https://github.com/pdavlin/arcitems/releases/download/v${VERSION}/checksums.txt | grep arcitems-windows-amd64.zip | awk '{print $1}')

# Update manifest
cd bucket
jq --arg version "$VERSION" \
   --arg url "https://github.com/pdavlin/arcitems/releases/download/v$VERSION/arcitems-windows-amd64.zip" \
   --arg hash "sha256:$CHECKSUM" \
   '.version = $version | .architecture."64bit".url = $url | .architecture."64bit".hash = $hash' \
   arcitems.json > temp.json && mv temp.json arcitems.json

git add arcitems.json
git commit -m "Update arcitems to $VERSION"
git push
```

## Step 5: Create Personal Access Token

1. Go to GitHub Settings > Developer settings > Personal access tokens > Fine-grained tokens
2. Click "Generate new token"
3. Token name: `scoop-arcitems-sync`
4. Repository access: Select "Only select repositories" > Choose `scoop-arcitems`
5. Permissions:
   - Contents: Read and write access
6. Generate token
7. Copy the token (you won't see it again)

## Step 6: Add Token to Main Repository

1. Go to `arcitems` repository > Settings > Secrets and variables > Actions
2. Click "New repository secret"
3. Name: `SCOOP_BUCKET_TOKEN`
4. Secret: Paste the token from Step 5
5. Click "Add secret"

## Step 7: Test Installation

On a Windows machine with Scoop installed:

```powershell
# Add the bucket
scoop bucket add pdavlin https://github.com/pdavlin/scoop-arcitems

# Install arcitems
scoop install arcitems

# Verify installation
arcitems --version
arcitems "broken flash"

# Test update detection
scoop update
scoop status
```

## Step 8: Trigger Excavator

The Excavator workflow will run automatically every 6 hours, but you can test it manually:

1. Go to Actions tab in the scoop-arcitems repository
2. Select "Excavator" workflow
3. Click "Run workflow"
4. Wait for it to complete
5. Check if the manifest was updated

## Troubleshooting

### Excavator Fails

If Excavator fails to update the manifest:
- Check the workflow logs in the Actions tab
- Verify the `autoupdate` section in the manifest is correct
- Ensure the checksums.txt file exists in the release
- The main repository workflow will also update the manifest as a backup

### Hash Mismatch

If Scoop reports a hash mismatch:
- Verify the SHA256 in the manifest matches checksums.txt
- Re-download the release file and recalculate the hash
- Update the manifest manually if needed

### Installation Fails

If `scoop install arcitems` fails:
- Check that the release file exists at the URL in the manifest
- Verify the binary name matches what's in the zip file
- Check the Scoop debug logs: `scoop install arcitems -d`
