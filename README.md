<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>How to Use Same EC2 Key Pair Across AWS Regions</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      line-height: 1.7;
      margin: 0 auto;
      max-width: 960px;
      padding: 2rem;
      background-color: #f9f9f9;
    }
    h1, h2, h3 {
      color: #333;
    }
    pre, code {
      background-color: #eee;
      padding: 0.5rem;
      border-radius: 4px;
      display: block;
      white-space: pre-wrap;
      word-wrap: break-word;
    }
    .image-placeholder {
      margin: 1rem 0;
      text-align: center;
      font-style: italic;
      color: #777;
      background: #fff;
      padding: 1rem;
      border: 1px dashed #ccc;
    }
  </style>
</head>
<body>
  <h1>How to Use the Same EC2 Key Pair Across Multiple AWS Regions — Step-by-Step Guide by Vishwanath Gowda</h1>

  <p>Working on multi-region AWS deployments and struggling to access your EC2 instances because of region-specific key pairs? Don’t worry! In this blog, I’ll show you how to use the same EC2 key pair in multiple AWS regions step by step, both manually and via the AWS CLI — so your workflow stays smooth and secure.</p>

  <h2>Why This Matters?</h2>
  <ul>
    <li>AWS key pairs are region-specific.</li>
    <li>A key pair you created in <code>us-east-1</code> won’t be available in <code>ap-south-1</code>.</li>
    <li>You’ll need to re-import it manually if you want to SSH into instances in another region.</li>
  </ul>

  <p>This blog will show:</p>
  <ul>
    <li>How to extract and reuse the key</li>
    <li>How to import it to other regions</li>
    <li>How to automate using AWS CLI</li>
    <li>Best security practices</li>
  </ul>

  <h2>Method 1: Manually Import the Key Pair to Each Region</h2>

  <h3>Step 1: Get the Private Key (.pem)</h3>
  <p>If you already downloaded the <code>.pem</code> file when creating the original key pair – you’re good to go.</p>
  <p>If not, AWS won’t let you download it again.</p>
  <strong>Important:</strong> Always store your <code>.pem</code> file securely!

  <h3>Step 2: Extract Public Key from Private Key (*.pem)</h3>
  <h4>Using <code>ssh-keygen</code> (Preferred for OpenSSH Format)</h4>
  <p><strong>Pre-requisites:</strong></p>
  <ul>
    <li>For Windows: PuTTYgen must be installed on your machine.</li>
    <li>For Mac/Linux: <code>ssh-keygen</code> or <code>openssl</code> should be available by default.</li>
  </ul>

  <p>If you are using a Mac or Linux machine, run either of these:</p>
  <pre><code>ssh-keygen -y -f my-key.pem &gt; my-key.pub</code></pre>
  <div class="image-placeholder">[Zoom image: ssh-keygen output shown]</div>

  <p>Output: <code>ssh-rsa AAAAB3... (OpenSSH format)</code></p>

  <h4>Using OpenSSL (PEM Format)</h4>
  <pre><code>openssl rsa -in my-key.pem -pubout &gt; my-key.pub</code></pre>
  <div class="image-placeholder">[Zoom image: OpenSSL output shown]</div>

  <p>Output: <code>-----BEGIN PUBLIC KEY-----...</code></p>

  <h3>Convert Between Key Formats</h3>
  <h4>PEM → OpenSSH Format</h4>
  <pre><code>ssh-keygen -i -m PKCS8 -f public-key-pem.pub &gt; public-key-openssh.pub</code></pre>

  <h4>OpenSSH → PEM Format</h4>
  <pre><code>ssh-keygen -e -m PEM -f public-key-openssh.pub &gt; public-key-pem.pub</code></pre>

  <h3>Verify Key Fingerprint (Match Check)</h3>
  <pre><code>openssl rsa -in my-key.pem -pubout | openssl md5
openssl pkey -in public-key-pem.pub -pubin | openssl md5</code></pre>
  <div class="image-placeholder">[Zoom image: Fingerprint output shown]</div>

  <h4>For Windows Users</h4>
  <p>Pre-requisite: PuTTYgen should be installed.</p>
  <p>If you have a <code>.ppk</code> file:</p>
  <pre><code>puttygen my-key.ppk -t rsa</code></pre>
  <p>This opens PuTTYgen with the public key. Copy it and paste into AWS:</p>
  <div class="image-placeholder">[Zoom image: PuTTYgen key display]</div>

  <h3>Step 3: Import Public Key to Another Region</h3>
  <ol>
    <li>Open AWS Console</li>
    <li>Switch to target region (e.g., ap-south-1)</li>
    <li>EC2 Dashboard → Key Pairs → Create Key Pair → Import Key Pair</li>
    <li>Enter name (e.g., <code>my-global-key</code>)</li>
    <li>Copy content of <code>my-key.pub</code> and paste</li>
    <li>Click Import</li>
  </ol>
  <p>✅ Done! You can now use the same key pair in this region.</p>

  <h3>Step 4: Repeat in Other Regions</h3>
  <p>Repeat the same steps for each AWS region. Now your <code>.pem</code> works globally!</p>

  <h2>Method 2: Import Key Pair Using AWS CLI</h2>

  <h3>Step 1: Extract Public Key Again (if needed)</h3>
  <pre><code>ssh-keygen -y -f my-key.pem &gt; my-key.pub</code></pre>

  <h3>Step 2: Import Using AWS CLI</h3>
  <pre><code>aws ec2 import-key-pair \
  --key-name "my-global-key" \
  --public-key-material "file://my-key.pub" \
  --region ap-south-1</code></pre>

  <h2>Important Notes</h2>
  <ul>
    <li>You can now SSH into any region's EC2 with the same <code>.pem</code></li>
    <li>AWS does not sync key pairs across regions</li>
    <li><strong>Never share</strong> your <code>.pem</code> file</li>
    <li>Use IAM roles + SSH agent forwarding for improved security</li>
    <li>Consider AWS Systems Manager Session Manager for keyless SSH</li>
  </ul>

  <h2>Pro Tip: Use AWS Secrets Manager</h2>

  <h4>Store the Key:</h4>
  <pre><code>aws secretsmanager create-secret \
  --name "global-ssh-key" \
  --secret-string "$(cat my-key.pem)"</code></pre>

  <h4>Retrieve Later:</h4>
  <pre><code>aws secretsmanager get-secret-value \
  --secret-id global-ssh-key \
  --region us-east-1</code></pre>

  <h2>Conclusion</h2>
  <p>AWS Key Pairs are region-specific, but with these steps, you can reuse the same key pair globally 🌍. Whether you're managing instances or automating infrastructure, this saves time and boosts efficiency while keeping your setup secure 🔐</p>

  <div class="image-placeholder">[Zoom image: Summary graphic]</div>

  <h2>About the Author</h2>
  <p><strong>Written by Vishwanath Gowda</strong><br>
  Cloud Security Engineer | DevSecOps Enthusiast | Trainer</p>
  <p>Follow me on Medium for more AWS, DevOps, and Security content 🔥<br>
  Reach out for project help, mentoring, or training collaborations!</p>
</body>
</html>
