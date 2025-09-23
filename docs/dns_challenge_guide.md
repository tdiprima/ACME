# 🌍 DNS Challenge Setup for Wildcard Certificates

DNS challenges are needed for wildcard certificates (*.example.com) and when port 80 isn't available.

## 🔧 Popular DNS Provider APIs

### Cloudflare
```bash
export CF_Token="your_cloudflare_api_token"
export CF_Account_ID="your_account_id" 

acme.sh --issue --dns dns_cf -d "*.example.com" -d "example.com"
```

### Route53 (AWS)
```bash
export AWS_ACCESS_KEY_ID="your_access_key"
export AWS_SECRET_ACCESS_KEY="your_secret_key"

acme.sh --issue --dns dns_aws -d "*.example.com" -d "example.com"
```

### Google Cloud DNS
```bash
# First, create service account and download JSON key
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account.json"

acme.sh --issue --dns dns_gcloud -d "*.example.com" -d "example.com"
```

### DigitalOcean
```bash
export DO_API_KEY="your_digitalocean_api_key"

acme.sh --issue --dns dns_dgon -d "*.example.com" -d "example.com"
```

### Namecheap
```bash
export NAMECHEAP_API_KEY="your_api_key"
export NAMECHEAP_USERNAME="your_username"

acme.sh --issue --dns dns_namecheap -d "*.example.com" -d "example.com"
```

### Manual DNS (Last Resort)
```bash
# This will give you a TXT record to create manually
acme.sh --issue --dns --yes-I-know-dns-manual-mode-enough-go-ahead-please \
    -d "*.example.com" -d "example.com"
```

## 🛠️ Full DNS Setup Script

```bash
#!/bin/bash
# DNS Challenge Setup for acme.sh

DOMAIN="example.com"
WILDCARD_DOMAIN="*.example.com"
EMAIL="admin@example.com"

# Choose your DNS provider and set credentials
DNS_PROVIDER="dns_cf"  # Change this: dns_cf, dns_aws, dns_gcloud, etc.

# Cloudflare example - replace with your provider's variables
export CF_Token="your_cloudflare_api_token_here"
export CF_Account_ID="your_account_id_here"

# Install acme.sh if not already installed
if [ ! -f ~/.acme.sh/acme.sh ]; then
    curl https://get.acme.sh | sh -s email=$EMAIL
    source ~/.bashrc
fi

# Issue wildcard certificate
~/.acme.sh/acme.sh --issue \
    --dns $DNS_PROVIDER \
    -d "$DOMAIN" \
    -d "$WILDCARD_DOMAIN"

# Install certificate (example for nginx)
~/.acme.sh/acme.sh --install-cert -d "$DOMAIN" \
    --key-file /etc/ssl/private/$DOMAIN.key \
    --fullchain-file /etc/ssl/certs/$DOMAIN.fullchain.pem \
    --reloadcmd "systemctl reload nginx"

echo "🎉 Wildcard certificate issued for $WILDCARD_DOMAIN!"
```

## 🔐 Security Best Practices

### 🔒 Limited DNS API Permissions
- Create API tokens with **minimal permissions**
- Only allow DNS record modification for your domains
- Set expiration dates on API tokens
- Use separate tokens for different domains

### 🏠 Separate Validation Server
- Run ACME client on a dedicated server
- Use API credentials only on that server  
- Copy certificates to web servers after issuance
- This limits blast radius if web server is compromised

### 📋 Example: Restricted Cloudflare Token
1. Go to Cloudflare → My Profile → API Tokens
2. Create Custom Token with these permissions:
   - Zone: Zone Settings: Read
   - Zone: Zone: Read  
   - Zone: DNS: Edit
3. Zone Resources: Include → Specific Zone → your-domain.com
4. Client IP Address Filtering: Include → Your server's IP

## ⚙️ Advanced Configuration

### Custom Hook Scripts
```bash
# Pre-hook: Run before issuing certificate
acme.sh --issue -d example.com --pre-hook "echo 'Starting certificate request'"

# Post-hook: Run after successful issuance  
acme.sh --issue -d example.com --post-hook "systemctl reload nginx"

# Renew-hook: Run only on successful renewal
acme.sh --issue -d example.com --renew-hook "/path/to/your/script.sh"
```

### Multiple Certificate Management
```bash
#!/bin/bash
# Manage multiple domains/certificates

DOMAINS=(
    "example.com,www.example.com"
    "*.api.example.com,api.example.com"  
    "blog.example.com"
)

for domain_set in "${DOMAINS[@]}"; do
    echo "🔐 Processing: $domain_set"
    
    # Convert comma-separated to -d arguments
    domain_args=""
    IFS=',' read -ra ADDR <<< "$domain_set"
    for domain in "${ADDR[@]}"; do
        domain_args="$domain_args -d $domain"
    done
    
    # Issue certificate
    acme.sh --issue --dns dns_cf $domain_args
    
    # Install certificate (customize per domain as needed)
    primary_domain="${ADDR[0]}"
    acme.sh --install-cert -d "$primary_domain" \
```

<br>
