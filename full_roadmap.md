# Stand Racing Self-Hosted Website Implementation Roadmap

## Infrastructure Foundation

### VPS Provider Selection
**Recommended Choice: Hetzner Cloud CPX21**
- **Cost**: €9.90/month (~$10.70)
- **Specs**: 3 vCPU, 4GB RAM, 80GB storage, 20TB bandwidth
- **Performance**: Excellent price-to-performance ratio, especially for EU audiences

**Alternative: Vultr High Frequency**
- **Cost**: $12/month
- **Specs**: 1 vCPU, 2GB RAM, 64GB NVMe, 2TB bandwidth
- **Benefits**: 32 global locations, superior NVMe performance

## Technology Stack Architecture

### Docker Compose Stack Configuration
The research reveals an optimized stack using **Caddy** for simplicity over Nginx:

```yaml
version: '3.8'
services:
  # FastAPI Application (Recommended for rapid development)
  app:
    build: ./app
    container_name: fastapi-app
    environment:
      - DATABASE_URL=mysql+aiomysql://${DB_USER}:${DB_PASSWORD}@db:3306/${DB_NAME}
    networks:
      - app-network
    depends_on:
      - db
    restart: unless-stopped

  # MariaDB optimized for small VPS
  db:
    image: mariadb:10.11
    container_name: mariadb
    command: >
      --innodb-buffer-pool-size=256M
      --max-connections=100
      --thread-pool-size=4
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - app-network
    restart: unless-stopped

  # Caddy for automatic HTTPS
  caddy:
    image: caddy:2-alpine
    container_name: caddy
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
    networks:
      - app-network
      - tunnel-network
    restart: unless-stopped

  # Cloudflare Tunnel for security
  tunnel:
    image: cloudflare/cloudflared:latest
    container_name: cloudflare-tunnel
    command: tunnel --no-autoupdate run --token ${CLOUDFLARE_TUNNEL_TOKEN}
    networks:
      - tunnel-network
    restart: unless-stopped
```

### Framework Recommendations
**Backend: FastAPI** (Python)
- Excellent for beginners with automatic documentation
- 40% fewer bugs due to type hints
- Simple Docker containerization
- High performance comparable to NodeJS

**Frontend Options:**
- **Next.js**: Full-stack JavaScript solution
- **React with Vite**: Lighter weight, faster development

## Discord Bot with Claude Integration

### Technology Choice
**discord.js** (Node.js) - Superior documentation and community support

### Implementation Strategy
```javascript
// Rate limiting for Claude API
class TokenBucket {
  constructor(capacity, refillRate) {
    this.capacity = capacity;
    this.tokens = capacity;
    this.refillRate = refillRate;
  }
  
  consume(tokens = 1) {
    this.refill();
    if (this.tokens >= tokens) {
      this.tokens -= tokens;
      return true;
    }
    return false;
  }
}

// Membership tier structure
const MEMBERSHIP_TIERS = {
  FREE: { level: 0, dailyQueries: 5 },
  BASIC: { level: 1, dailyQueries: 25 },
  PREMIUM: { level: 2, dailyQueries: 100 },
  EXPERT: { level: 3, dailyQueries: 500 }
};
```

## Security Implementation

### Essential Security Hardening
```bash
# 1. Install security packages
apt install ufw fail2ban aide rkhunter unattended-upgrades

# 2. Configure UFW firewall
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp  # SSH on custom port
ufw allow 80/tcp  # HTTP
ufw allow 443/tcp # HTTPS
ufw enable

# 3. SSH hardening in /etc/ssh/sshd_config
Port [custom-port]
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
```

### Automated Backup Strategy
```yaml
backup:
  image: offen/docker-volume-backup:v2
  environment:
    - BACKUP_CRON_EXPRESSION=0 2 * * *
    - BACKUP_RETENTION_DAYS=30
  volumes:
    - data:/backup/data:ro
    - /var/run/docker.sock:/var/run/docker.sock:ro
```

## YouTube Membership Integration

### Primary Strategy: Discord Synchronization
YouTube provides official Discord integration for membership verification:
- Automatic role assignment
- Real-time synchronization
- Multiple tier support

### Database Schema for Tracking
```sql
CREATE TABLE member_verification (
    user_id BIGINT PRIMARY KEY,
    discord_id BIGINT UNIQUE,
    youtube_channel_id VARCHAR(255),
    membership_level VARCHAR(100),
    verification_status ENUM('pending', 'verified', 'expired'),
    verified_at TIMESTAMP
);
```

## Email Configuration

### Recommended Approach
For simplicity, use **SMTP relay services** instead of self-hosted:
- **SendGrid**: 100 emails/day free
- **Mailgun**: 10,000 emails/month free trial
- **Amazon SES**: $0.10 per 1,000 emails

If self-hosting is required:
```bash
# Basic Postfix configuration
apt install postfix
# Configure for internet site
# Set up SPF, DKIM, DMARC records
```

## 7-Day Implementation Timeline

### Day 1-2: Infrastructure & Security (32 hours)
**Day 1 Focus:**
- Hours 1-2: VPS setup, Docker installation
- Hours 3-5: Database and backend foundation
- Hours 6-8: Security implementation (UFW, fail2ban, SSH)
- Hours 9-11: Basic API endpoints (auth, users)
- Hours 12-14: Testing setup
- Hours 15-16: Local deployment testing

**Day 2 Focus:**
- Hours 17-19: Frontend foundation setup
- Hours 20-22: Authentication UI
- Hours 23-25: Post system backend
- Hours 26-28: Post system frontend
- Hours 29-31: CI/CD pipeline
- Hour 32: Complete system test

### Day 3-4: Core Application (32 hours)
**Day 3:**
- Advanced post features (categories, search)
- User profile system
- Community features (comments, reactions)
- Database optimization
- Frontend polish

**Day 4:**
- Admin panel foundation
- Content management system
- API security hardening
- Performance optimization
- Integration testing

### Day 5-6: Discord Bot & Claude AI (32 hours)
**Day 5:**
- Discord bot setup using boilerplate
- Website-Discord integration
- Core bot features
- Claude API preparation
- Database extensions

**Day 6:**
- Claude AI implementation
- Advanced Discord features
- Real-time features (WebSockets)
- Integration testing
- Security and performance tuning

### Day 7: Testing & Deployment (16 hours)
- Comprehensive testing
- Production deployment
- Documentation creation
- Launch preparation
- Go-live and monitoring

## NextCloud Integration Options

For file storage and collaboration:
- WebDAV API for file management
- Member-only folders with tiered access
- Calendar integration for project scheduling
- Collaborative documentation features

## Budget Breakdown

**Monthly Costs:**
- VPS Hosting: ~$10-12/month
- Domain: Already owned
- Claude API: Usage-based (~$5-20/month depending on usage)
- **Total**: ~$15-32/month

**One-time Setup:**
- Development time: 112 hours over 7 days
- No additional software costs (all open-source)

## Critical Success Factors

1. **Use proven templates**: TaitoUnited/full-stack-template for rapid development
2. **Prioritize MVP features**: Authentication, posts, Discord bot, basic admin
3. **Automate everything possible**: CI/CD, backups, monitoring
4. **Security first**: Implement security measures from day one
5. **Buffer time**: 20% buffer built into timeline for learning/troubleshooting

## Quick Start Commands

```bash
# 1. Initial VPS setup
ssh root@your-vps-ip
apt update && apt upgrade -y
apt install docker.io docker-compose git ufw fail2ban

# 2. Clone your repository
git clone https://github.com/yourusername/stand-racing-site.git
cd stand-racing-site

# 3. Configure environment
cp .env.example .env
nano .env  # Add your secrets

# 4. Launch stack
docker-compose up -d

# 5. Configure Cloudflare tunnel
# Follow Cloudflare dashboard instructions
```

This roadmap provides a clear path from zero to a fully functional self-hosted community website with Discord bot integration and Claude AI capabilities, all within the constraints of a 7-day timeline and minimal budget. The emphasis on security, automation, and proven technologies ensures a maintainable and scalable solution for the Stand Racing YouTube channel community.