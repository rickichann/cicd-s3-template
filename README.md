# Python Script S3 Deployment - CI/CD Pipeline

Automatically deploy Python scripts and folders to AWS S3 using GitHub Actions.

---

## How It Works

Every time you push code to the `main` branch:
1. GitHub Actions fetches all files from the repository
2. Authenticates with AWS using stored credentials
3. Uploads all files and folders to your S3 bucket
4. Maintains the same folder structure in S3

---

## Setup Instructions

### 1. AWS Configuration

Create an S3 bucket and IAM user with proper permissions.

**Required IAM Policy:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
        },
        {
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME"
        }
    ]
}
```

### 2. GitHub Secrets

Add these secrets in your repository: **Settings → Secrets and variables → Actions**

| Secret Name | Description | Example |
|------------|-------------|---------|
| `AWS_ACCESS_KEY_ID` | Your AWS access key ID | `AKIAIOSFODNN7EXAMPLE` |
| `AWS_SECRET_ACCESS_KEY` | Your AWS secret access key | (40 character string) |
| `S3_BUCKET_NAME` | S3 bucket name only | `my-deployment-bucket` |

**Important:** Use bucket name only, not `s3://bucket-name` or full URL.

### 3. Deploy Your Code

```bash
# Add your changes
git add .

# Commit with a message
git commit -m "Update scripts"

# Push to trigger deployment
git push origin main
```

### 4. Monitor Deployment

1. Go to your GitHub repository
2. Click the **Actions** tab
3. View the running workflow
4. Check logs for any errors
5. Verify files in your S3 bucket

---

## Project Structure

```
.
├── .github/
│   └── workflows/
│       └── deploy-to-s3.yml    # CI/CD configuration
├── pipelines/
│   └── rp.py                    # Python scripts
├── deploy_script.py             # Main script
└── README.md
```

---

## What Gets Deployed

The workflow uploads **all files** except:
- `.git/` folder
- `.github/` folder  
- `README.md` file

To modify exclusions, edit `.github/workflows/deploy-to-s3.yml`

---

## Troubleshooting

### Credentials could not be loaded
- Verify all three secrets are added with exact names (case-sensitive)
- Check for typos or extra spaces

### AccessDenied error
- IAM user lacks S3 permissions
- Add the required policy shown above

### Invalid bucket name
- Use bucket name only in `S3_BUCKET_NAME`
- Remove `s3://` prefix or any URLs

---

## Manual Deployment

Deploy without CI/CD using AWS CLI:

```bash
# Sync all files
aws s3 sync . s3://your-bucket-name/ --exclude ".git/*" --exclude ".github/*"

# Upload single file
aws s3 cp deploy_script.py s3://your-bucket-name/deploy_script.py
```

---

## Customization

### Change AWS Region

Edit `.github/workflows/deploy-to-s3.yml`:
```yaml
aws-region: us-east-1  # Change to your region
```

### Change Trigger Branch

Edit `.github/workflows/deploy-to-s3.yml`:
```yaml
on:
  push:
    branches:
      - main  # Change to your branch
```

### Manual Trigger

Go to **Actions** tab → **Deploy to S3** → **Run workflow**

---

