
# SYSTEM CONFIGURATION REVIEW

**Document Version:** 1.0  
**Last Updated:** 2026-03-23  
**Status:** Complete



## TABLE OF CONTENTS

1. [Executive Summary](#executive-summary)
2. [Configuration Files Overview](#configuration-files-overview)
3. [Environment Variables Reference](#environment-variables-reference)
4. [Application Configuration](#application-configuration)
5. [Database Configuration](#database-configuration)
6. [Cache & Session Configuration](#cache--session-configuration)
7. [Authentication & Authorization Configuration](#authentication--authorization-configuration)
8. [Logging & Audit Configuration](#logging--audit-configuration)
9. [Build & Asset Configuration](#build--asset-configuration)
10. [Docker & Infrastructure Configuration](#docker--infrastructure-configuration)
11. [Security Configuration](#security-configuration)
12. [Development vs Production Setup](#development-vs-production-setup)
13. [Quick Start Configuration Checklist](#quick-start-configuration-checklist)
14. [Troubleshooting Configuration Issues](#troubleshooting-configuration-issues)

---

## EXECUTIVE SUMMARY

The Health Data Bank system uses Laravel 11 with a comprehensive configuration structure that manages:
- **Application Settings** (name, environment, timezone, locale)
- **Database Connections** (MySQL with multiple connection options)
- **Authentication** (Laravel Sanctum, Fortify, 2FA)
- **Authorization** (Spatie Permission for role-based access)
- **Caching & Sessions** (Database-backed by default)
- **Logging & Auditing** (Centralized audit trail for HIPAA compliance)
- **Build Tools** (Vite, Tailwind, PostCSS)
- **Docker Infrastructure** (Laravel Sail with MySQL container)

All configuration is managed through:
- **Environment variables** (`.env` file)
- **Configuration files** (in `config/` directory)
- **Docker Compose** (infrastructure setup)

---

## CONFIGURATION FILES OVERVIEW

### ROOT LEVEL CONFIGURATION FILES

| File | Purpose | Type |
|------|---------|------|
| `.env` | Environment variables (sensitive data) | Env File |
| `.env.example` | Template for `.env` configuration | Env Template |
| `.editorconfig` | Code editor formatting standards | EditorConfig |
| `.gitignore` | Files to exclude from version control | Git Config |
| `.gitattributes` | Git attributes for line endings | Git Config |
| `compose.yaml` | Docker Compose infrastructure setup | YAML |
| `composer.json` | PHP dependencies | JSON |
| `composer.lock` | Locked PHP dependency versions | JSON |
| `package.json` | Node.js dependencies | JSON |
| `package-lock.json` | Locked Node.js dependency versions | JSON |
| `phpunit.xml` | Testing framework configuration | XML |
| `vite.config.js` | Frontend build tool configuration | JavaScript |
| `postcss.config.js` | CSS post-processing configuration | JavaScript |
| `tailwind.config.js` | Tailwind CSS customization | JavaScript |

### CONFIGURATION DIRECTORY (`config/`)

The `config/` directory contains Laravel configuration files:

| File | Purpose |
|------|---------|
| `app.php` | Application settings (name, env, debug, URL, timezone, locale, encryption) |
| `auth.php` | Authentication guards, providers, password reset |
| `cache.php` | Cache stores configuration (database, file, Redis, etc.) |
| `database.php` | Database connections (MySQL, PostgreSQL, etc.) |
| `filesystem.php` | File storage configuration |
| `logging.php` | Application logging channels and drivers |
| `mail.php` | Email service configuration (SMTP, Mailgun, SES, etc.) |
| `queue.php` | Job queue configuration (database, Redis, SQS, etc.) |
| `services.php` | Third-party services (AWS, Slack, Postmark, etc.) |
| `session.php` | Session management configuration |
| `audit.php` | Auditing configuration (owen-it/laravel-auditing) |
| `permission.php` | Role-based permission system (Spatie) |
| `fortify.php` | Authentication features (2FA, registration, password reset) |
| `jetstream.php` | Team and organization features |
| `backup.php` | Database backup configuration |

---

## ENVIRONMENT VARIABLES REFERENCE

### CORE APPLICATION VARIABLES

```bash
# Application Name & Environment
APP_NAME="Health Data Bank"          # Application display name
APP_ENV=local                        # Environment: local, staging, production
APP_DEBUG=true                       # Enable/disable debug mode (false in production)
APP_URL=http://localhost            # Application URL for CLI operations

# Application Key (Encryption)
APP_KEY=base64:xxxxx...             # Encryption key (generate with: php artisan key:generate)
APP_CIPHER=AES-256-CBC              # Encryption cipher

# Localization
APP_LOCALE=en                        # Default locale
APP_FALLBACK_LOCALE=en               # Fallback locale if primary not found
APP_TIMEZONE=UTC                     # Application timezone
```

**Notes:**
- `APP_KEY` is CRITICAL and must be kept secret
- `APP_DEBUG=true` should NEVER be enabled in production (exposes sensitive information)
- `APP_ENV` controls which configuration is loaded

### DATABASE VARIABLES

```bash
# MySQL Database Connection
DB_CONNECTION=mysql                 # Database driver: mysql, pgsql, sqlite, sqlsrv
DB_HOST=mysql                       # Database host (mysql container in Docker)
DB_PORT=3306                        # Database port
DB_DATABASE=health_data_bank        # Database name
DB_USERNAME=health_user             # Database user
DB_PASSWORD=password                # Database password

# Optional: Secondary Connections
DB_CACHE_CONNECTION=mysql           # Cache table connection
DB_QUEUE_CONNECTION=mysql           # Queue table connection
```

**Notes:**
- In Docker, `DB_HOST=mysql` (container name)
- In local development, `DB_HOST=127.0.0.1` or `localhost`
- Always use strong passwords in production

### CACHE CONFIGURATION

```bash
# Cache Store (default: database)
CACHE_STORE=database                # Cache driver: array, database, file, redis, memcached
DB_CACHE_TABLE=cache                # Cache table name
DB_CACHE_CONNECTION=mysql           # Cache connection

# Optional: Redis Cache
REDIS_CLIENT=phpredis               # Redis client: phpredis, predis
REDIS_CACHE_CONNECTION=cache        # Redis cache connection name
REDIS_HOST=127.0.0.1                # Redis host
REDIS_PASSWORD=null                 # Redis password
REDIS_PORT=6379                     # Redis port
```

**Supported Drivers:**
- `database` - Stores cache in database table (default, no extra setup)
- `file` - Stores cache in filesystem
- `redis` - High-performance in-memory cache
- `memcached` - Distributed memory caching
- `array` - In-memory array (per-request, no persistence)

### SESSION CONFIGURATION

```bash
# Session Management
SESSION_DRIVER=database             # Session store: file, cookie, database, redis
SESSION_LIFETIME=120                # Session timeout in minutes
SESSION_COOKIE=health_bank_session  # Session cookie name
SESSION_PATH=/                      # Cookie path
SESSION_DOMAIN=.example.com         # Cookie domain (for subdomain sharing)
SESSION_SECURE=false                # HTTPS only cookies (true in production)
SESSION_HTTP_ONLY=true              # HTTP-only cookies (prevent JavaScript access)
SESSION_SAME_SITE=lax               # CSRF protection: strict, lax, none
```

### QUEUE CONFIGURATION

```bash
# Job Queue
QUEUE_CONNECTION=database           # Queue driver: database, redis, sqs, sync
DB_QUEUE_TABLE=jobs                 # Queue table
DB_QUEUE=default                    # Default queue name
DB_QUEUE_RETRY_AFTER=90             # Retry failed jobs after (seconds)

# Optional: Redis Queue
REDIS_QUEUE_CONNECTION=default      # Redis queue connection
REDIS_QUEUE=default                 # Redis queue name
```

**Drivers:**
- `database` - Process jobs from database (good for small apps)
- `redis` - High-performance queue (recommended for production)
- `sync` - Process synchronously (good for development)

### MAIL CONFIGURATION

```bash
# Mail Service
MAIL_MAILER=smtp                    # Mailer: smtp, mailgun, postmark, ses, resend, log, array
MAIL_HOST=smtp.mailtrap.io          # SMTP host
MAIL_PORT=2525                      # SMTP port
MAIL_USERNAME=xxxx                  # SMTP username
MAIL_PASSWORD=xxxx                  # SMTP password
MAIL_ENCRYPTION=tls                 # Encryption: tls, ssl
MAIL_FROM_ADDRESS=noreply@example.com # From email address
MAIL_FROM_NAME="Health Data Bank"   # From display name

# Optional: Third-party Services
POSTMARK_API_KEY=                   # Postmark API key
RESEND_API_KEY=                     # Resend API key
MAILGUN_DOMAIN=                     # Mailgun domain
MAILGUN_SECRET=                     # Mailgun secret
AWS_ACCESS_KEY_ID=                  # AWS access key
AWS_SECRET_ACCESS_KEY=              # AWS secret key
AWS_DEFAULT_REGION=us-east-1        # AWS region
```

### LOGGING CONFIGURATION

```bash
# Application Logging
LOG_CHANNEL=stack                   # Log channel: stack, single, daily, slack
LOG_LEVEL=debug                     # Log level: debug, info, notice, warning, error, critical, alert, emergency
LOG_DAILY_DAYS=14                   # Keep daily logs for X days

# Optional: Slack Logging
LOG_SLACK_WEBHOOK_URL=              # Slack webhook URL for error notifications
LOG_SLACK_USERNAME=Laravel Log      # Slack bot username
LOG_SLACK_EMOJI=:boom:              # Slack message emoji
```

### AUTHENTICATION & AUTHORIZATION

```bash
# Authentication Defaults
AUTH_GUARD=web                      # Default auth guard: web, api
AUTH_PASSWORD_BROKER=users          # Password reset broker
AUTH_PASSWORD_TIMEOUT=10800         # Password confirmation timeout (seconds)

# Fortify (Authentication Features)
FORTIFY_GUARD=web                   # Fortify guard
FORTIFY_REDIRECTS_TO=/dashboard     # Redirect after login
FORTIFY_FEATURES=registration,reset_passwords,update_profile_information,update_passwords,two_factor_authentication

# Jetstream (Teams)
JETSTREAM_STACK=livewire            # Stack: livewire, inertia
JETSTREAM_FEATURES=profile_photos,api,teams
```

### BACKUP CONFIGURATION

```bash
# Database Backups
BACKUP_ENABLED=true                 # Enable/disable backups
BACKUP_STORE_LOCATION=s3            # Backup storage: local, s3, gcs
BACKUP_COMPRESSION=gzip             # Compression: gzip, bzip2
```

### DEVELOPMENT TOOLS

```bash
# Debugging
DEBUGBAR_ENABLED=true               # Laravel Debugbar (development only)

# Testing
APP_ENV_TEST=testing                # Test environment
CACHE_STORE_TEST=array              # Test cache store
```

---

## APPLICATION CONFIGURATION

### FILE: `config/app.php`

**Purpose:** Core application settings

**Key Settings:**

```php
return [
    // Application name used in notifications and UI
    'name' => env('APP_NAME', 'Health Data Bank'),
    
    // Environment: local, staging, production
    'env' => env('APP_ENV', 'production'),
    
    // Debug mode (disable in production!)
    'debug' => (bool) env('APP_DEBUG', false),
    
    // Base URL for CLI commands
    'url' => env('APP_URL', 'http://localhost'),
    
    // Timezone for all date operations
    'timezone' => 'UTC',
    
    // Default locale
    'locale' => env('APP_LOCALE', 'en'),
    'fallback_locale' => env('APP_FALLBACK_LOCALE', 'en'),
    
    // Encryption cipher and key
    'cipher' => 'AES-256-CBC',
    'key' => env('APP_KEY'),
    
    // Service providers to load
    'providers' => [
        // ... (Laravel and package providers)
    ],
    
    // Service aliases
    'aliases' => [
        // ... (Class aliases)
    ],
];
```

**Configuration Details:**

| Setting | Purpose | Values |
|---------|---------|--------|
| `env` | Environment type | `local`, `staging`, `production` |
| `debug` | Show error details | `true` (dev), `false` (prod) |
| `timezone` | Date/time operations | `UTC`, `America/New_York`, etc. |
| `locale` | Default language | `en`, `es`, `fr`, etc. |
| `cipher` | Encryption algorithm | `AES-256-CBC` (recommended) |

---

## DATABASE CONFIGURATION

### FILE: `config/database.php`

**Purpose:** Database connection settings

**MySQL Connection (Default):**

```php
'mysql' => [
    'driver' => 'mysql',
    'url' => env('DATABASE_URL'),
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', 3306),
    'database' => env('DB_DATABASE', 'forge'),
    'username' => env('DB_USERNAME', 'forge'),
    'password' => env('DB_PASSWORD', ''),
    'unix_socket' => env('DB_SOCKET', ''),
    'charset' => 'utf8mb4',        // Character set (supports emoji)
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',                // Table name prefix
    'prefix_indexes' => true,
    'strict' => true,              // Strict mode
    'engine' => null,              // InnoDB (default) or MyISAM
    'options' => extension_loaded('pdo_mysql') ? array_filter([
        PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
    ]) : [],
],
```

**Key Settings:**

| Setting | Purpose | Recommended |
|---------|---------|-------------|
| `charset` | Character encoding | `utf8mb4` (full Unicode) |
| `collation` | Character collation | `utf8mb4_unicode_ci` |
| `strict` | Strict SQL mode | `true` (enforces constraints) |
| `engine` | Storage engine | `InnoDB` (default, transactions) |

**Connection Testing:**

```bash
# Test MySQL connection
php artisan migrate:status

# Test database connectivity
php artisan tinker
>>> DB::connection()->getPDO();
```

---

## CACHE & SESSION CONFIGURATION

### CACHE CONFIGURATION (`config/cache.php`)

**Default: Database Cache**

```php
'default' => env('CACHE_STORE', 'database'),

'stores' => [
    'database' => [
        'driver' => 'database',
        'connection' => env('DB_CACHE_CONNECTION'),
        'table' => env('DB_CACHE_TABLE', 'cache'),
    ],
    
    'redis' => [
        'driver' => 'redis',
        'connection' => env('REDIS_CACHE_CONNECTION', 'cache'),
    ],
    
    'file' => [
        'driver' => 'file',
        'path' => storage_path('framework/cache/data'),
    ],
],

'prefix' => env('CACHE_PREFIX', 'health_data_bank_cache'),
```

**Cache Clearing Commands:**

```bash
# Clear all cache
php artisan cache:clear

# Clear specific cache key
php artisan cache:forget key_name

# Clear app cache from code
Cache::flush();
```

### SESSION CONFIGURATION (`config/session.php`)

**Default: Database Sessions**

```php
'driver' => env('SESSION_DRIVER', 'database'),
'lifetime' => env('SESSION_LIFETIME', 120),  // 120 minutes
'expire_on_close' => false,

'cookie' => env(
    'SESSION_COOKIE',
    Str::slug(env('APP_NAME')).'_session'
),

'path' => env('SESSION_PATH', '/'),
'domain' => env('SESSION_DOMAIN'),
'secure' => env('SESSION_SECURE', false),    // HTTPS only
'http_only' => env('SESSION_HTTP_ONLY', true),
'same_site' => env('SESSION_SAME_SITE', 'lax'),
```

**Session Cleanup:**

```bash
# Clear expired sessions (run periodically)
php artisan session:prune-stale-files
```

---

## AUTHENTICATION & AUTHORIZATION CONFIGURATION

### AUTHENTICATION (`config/auth.php`)

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'sanctum' => [
        'driver' => 'sanctum',
        'provider' => 'users',
    ],
],

'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\User::class,
    ],
],

'passwords' => [
    'users' => [
        'provider' => 'users',
        'table' => 'password_reset_tokens',
        'expire' => 60,      // Token expiration (minutes)
        'throttle' => 60,    // Rate limit (seconds)
    ],
],

'password_timeout' => 10800,  // Confirm password timeout (3 hours)
```

### FORTIFY CONFIGURATION (`config/fortify.php`)

**2FA and Authentication Features:**

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::updateProfileInformation(),
    Features::updatePasswords(),
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
    ]),
],

'guard' => 'sanctum',
'passwords' => 'users',
'redirect' => 'dashboard',
```

### PERMISSION CONFIGURATION (`config/permission.php`)

**Spatie Permission Settings:**

```php
'models' => [
    'permission' => App\Models\Permission::class,
    'role' => App\Models\Role::class,
],

'table_names' => [
    'roles' => 'roles',
    'permissions' => 'permissions',
    'model_has_permissions' => 'model_has_permissions',
    'model_has_roles' => 'model_has_roles',
    'role_has_permissions' => 'role_has_permissions',
],

'column_names' => [
    'role_pivot_key' => 'role_id',
    'permission_pivot_key' => 'permission_id',
    'model_morph_key' => 'model_id',
],

'enable_wildcard_permission' => false,
'display_permission_in_exception' => false,
'display_role_in_exception' => false,
```

**System Roles:**
- `user` - Regular patient user
- `researcher` - Research data access
- `provider` - Healthcare provider
- `admin` - System administrator

---

## LOGGING & AUDIT CONFIGURATION

### LOGGING (`config/logging.php`)

**Default: Stack Channel**

```php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['single'],
        'ignore_exceptions' => false,
    ],
    
    'single' => [
        'driver' => 'single',
        'path' => storage_path('logs/laravel.log'),
        'level' => env('LOG_LEVEL', 'debug'),
    ],
    
    'daily' => [
        'driver' => 'daily',
        'path' => storage_path('logs/laravel.log'),
        'level' => env('LOG_LEVEL', 'debug'),
        'days' => env('LOG_DAILY_DAYS', 14),
    ],
    
    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'level' => env('LOG_LEVEL', 'critical'),
    ],
],
```

**Log Levels (priority):**
1. `emergency` - System is unusable
2. `alert` - Action must be taken immediately
3. `critical` - Critical conditions
4. `error` - Error conditions
5. `warning` - Warning conditions
6. `notice` - Normal but significant
7. `info` - Informational messages
8. `debug` - Debug-level messages

**Log Management:**

```bash
# View logs
tail -f storage/logs/laravel.log

# Clear logs
rm storage/logs/*.log

# Log rotation (daily)
# Automatic with 'daily' driver, keeps 14 days
```

### AUDIT CONFIGURATION (`config/audit.php`)

**Owen-it/Laravel-Auditing:**

```php
'enabled' => env('AUDITING_ENABLED', true),

'events' => [
    'created',
    'updated',
    'deleted',
    'restored',
],

'user' => [
    'morph_prefix' => 'user',
    'guards' => ['web', 'api'],
],

'timestamps' => false,        // Don't audit timestamp changes
'empty_values' => true,       // Record empty value changes
'allowed_array_values' => false,  // Don't audit array values
'threshold' => 0,             // No limit on audit records
```

**Audit Logging:**

```php
// In AuditLogger service: Never log sensitive data
AuditLogger::log(
    'user_login',
    ['auth', 'success'],
    null,
    [],
    ['user_id' => $userId]  // IDs only, never passwords
);
```

---

## BUILD & ASSET CONFIGURATION

### VITE CONFIGURATION (`vite.config.js`)

**Frontend Build Tool:**

```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js'
            ],
            refresh: true,  // Hot reload in development
        }),
    ],
});
```

**Vite Commands:**

```bash
# Development (with hot reload)
npm run dev

# Production build
npm run build

# Preview production build
npm run preview
```

### TAILWIND CSS CONFIGURATION (`tailwind.config.js`)

**CSS Framework:**

```javascript
export default {
    content: [
        './resources/views/**/*.blade.php',
        './resources/js/**/*.js',
    ],
    theme: {
        extend: {
            colors: {
                // Custom colors
            },
        },
    },
    plugins: [
        // Plugins
    ],
};
```

### POSTCSS CONFIGURATION (`postcss.config.js`)

**CSS Post-Processing:**

```javascript
export default {
    plugins: {
        tailwindcss: {},
        autoprefixer: {},
    },
};
```

### EDITOR CONFIGURATION (`.editorconfig`)

**Code Formatting Standards:**

```ini
[*]
charset = utf-8
end_of_line = lf
indent_size = 4
indent_style = space
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false

[*.{yml,yaml}]
indent_size = 2
```

---

## DOCKER & INFRASTRUCTURE CONFIGURATION

### DOCKER COMPOSE (`compose.yaml`)

**Multi-container Application Stack:**

```yaml
services:
  laravel.test:
    build:
      context: .
      dockerfile: Dockerfile
    image: health-data-bank:latest
    ports:
      - "80:80"
      - "443:443"
    environment:
      - DB_HOST=mysql
      - DB_DATABASE=health_data_bank
      - DB_USERNAME=health_user
      - DB_PASSWORD=${DB_PASSWORD:-secret}
    volumes:
      - .:/var/www/html
    depends_on:
      - mysql
    networks:
      - health-bank-network

  mysql:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: health_data_bank
      MYSQL_USER: health_user
      MYSQL_PASSWORD: ${DB_PASSWORD:-secret}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD:-root}
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - health-bank-network

volumes:
  mysql_data:
    driver: local

networks:
  health-bank-network:
    driver: bridge
```

**Docker Commands:**

```bash
# Start containers
docker-compose -f compose.yaml up -d

# Stop containers
docker-compose down

# View logs
docker-compose logs -f laravel.test

# Execute commands in container
docker-compose exec laravel.test php artisan migrate
```

### ENVIRONMENT SETUP FOR DOCKER

**Development (.env):**

```bash
APP_NAME="Health Data Bank"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost

DB_HOST=mysql              # Docker container name
DB_DATABASE=health_data_bank
DB_USERNAME=health_user
DB_PASSWORD=secret
```

**Production (.env.production):**

```bash
APP_NAME="Health Data Bank"
APP_ENV=production
APP_DEBUG=false            # IMPORTANT: Never true in production
APP_URL=https://healthdatabank.com

DB_HOST=prod-mysql-host
DB_DATABASE=health_data_bank_prod
DB_USERNAME=hdb_prod_user
DB_PASSWORD=strong-password-here
```

---

## SECURITY CONFIGURATION

### HTTPS & SSL

**Production Environment (.env):**

```bash
APP_URL=https://yourdomain.com
SESSION_SECURE=true        # HTTPS-only cookies
SESSION_HTTP_ONLY=true     # Prevent JavaScript access
```

**NGINX Configuration (example):**

```nginx
server {
    listen 443 ssl http2;
    server_name yourdomain.com;
    
    ssl_certificate /path/to/cert.crt;
    ssl_certificate_key /path/to/key.key;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    root /var/www/html/public;
    index index.php;
    
    location ~ \.php$ {
        fastcgi_pass unix:/run/php-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$server_name$request_uri;
}
```

### CSRF PROTECTION

**Automatic via Middleware:**

```php
// In requests, include CSRF token
<input type="hidden" name="_token" value="{{ csrf_token() }}">

// In AJAX requests
headers: {
    'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content
}
```

### API TOKEN SECURITY

**Sanctum Configuration (.env):**

```bash
# API token scopes
SANCTUM_STATEFUL_DOMAINS=localhost:3000,example.com
SANCTUM_ENCRYPT_COOKIES=true
```

**Token Usage:**

```php
// Create token with scopes
$token = $user->createToken('api-token', ['read', 'create']);

// Use token in API requests
curl -H "Authorization: Bearer $token" https://api.example.com/data
```

### ENVIRONMENT VARIABLE SECURITY

**Never Commit Secrets:**

```bash
# .gitignore
.env                    # Never commit
.env.*.local           # Never commit
storage/logs/*         # Never commit
node_modules/          # Never commit
vendor/                # Never commit

# Safe to commit
.env.example           # Template only
```

### ENCRYPTION

**Application Encryption:**

```php
// Encrypt sensitive data
$encrypted = encrypt('sensitive-data');
$decrypted = decrypt($encrypted);

// Database encryption (fields)
// In migration:
Schema::table('users', function (Blueprint $table) {
    $table->encrypted('ssn');  // Automatically encrypted
});
```

---

## DEVELOPMENT VS PRODUCTION SETUP

### DEVELOPMENT ENVIRONMENT

**.env (Development):**

```bash
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=mysql           # Docker container
DB_DATABASE=health_data_bank
DB_USERNAME=health_user
DB_PASSWORD=secret

CACHE_STORE=database
LOG_CHANNEL=single
LOG_LEVEL=debug

MAIL_MAILER=log         # Log emails instead of sending
```

**Development Commands:**

```bash
# Start Docker containers
docker-compose up -d

# Run migrations
php artisan migrate

# Seed database
php artisan db:seed

# Start development server with hot reload
npm run dev

# View logs
tail -f storage/logs/laravel.log
```

### PRODUCTION ENVIRONMENT

**.env.production:**

```bash
APP_ENV=production
APP_DEBUG=false         # CRITICAL: Always false
APP_URL=https://yourdomain.com

DB_CONNECTION=mysql
DB_HOST=prod-db-host
DB_DATABASE=health_data_bank_prod
DB_USERNAME=secure_user
DB_PASSWORD=strong-password

CACHE_STORE=redis       # Use Redis for better performance
SESSION_SECURE=true     # HTTPS only
SESSION_HTTP_ONLY=true

LOG_CHANNEL=stack
LOG_LEVEL=warning       # Only log warnings and errors

MAIL_MAILER=smtp        # Use real mail service
MAIL_HOST=smtp.mailgun.org
MAIL_PORT=587
MAIL_USERNAME=postmaster
MAIL_PASSWORD=password

BACKUP_ENABLED=true
```

**Production Deployment:**

```bash
# Build assets
npm run build

# Install dependencies
composer install --no-dev --optimize-autoloader

# Run migrations
php artisan migrate --force

# Cache configuration
php artisan config:cache

# Optimize application
php artisan optimize

# Set permissions
chmod -R 755 storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache
```

### STAGING ENVIRONMENT

**Similar to production but with:**

```bash
APP_ENV=staging
APP_DEBUG=false         # Can be true for troubleshooting
LOG_LEVEL=info          # Log informational messages

# Test credentials (not production data)
DB_DATABASE=health_data_bank_staging
DB_PASSWORD=staging_password
```

---

## QUICK START CONFIGURATION CHECKLIST

### INITIAL SETUP CHECKLIST

- [ ] Clone repository: `git clone https://github.com/Health-Data-Bank-Team-1/health-data-bank.git`
- [ ] Create `.env` file: `cp .env.example .env`
- [ ] Generate APP_KEY: `docker-compose exec laravel.test php artisan key:generate`
- [ ] Configure database in `.env`:
  - [ ] `DB_HOST=mysql`
  - [ ] `DB_DATABASE=health_data_bank`
  - [ ] `DB_USERNAME=health_user`
  - [ ] `DB_PASSWORD=your-password`
- [ ] Start Docker: `docker-compose -f compose.yaml up -d`
- [ ] Run migrations: `docker-compose exec laravel.test php artisan migrate`
- [ ] Install Node dependencies: `npm install`
- [ ] Build frontend assets: `npm run build`
- [ ] Access application: `http://localhost`
- [ ] Verify API: `curl http://localhost/api/patients`

### CONFIGURATION VERIFICATION CHECKLIST

```bash
# 1. Database connection
php artisan tinker
>>> DB::connection()->getPDO();  # Should succeed

# 2. Cache functionality
>>> Cache::put('test', 'value');
>>> Cache::get('test');          # Should return 'value'

# 3. Session functionality
>>> Session::put('test', 'data');
>>> Session::get('test');        # Should return 'data'

# 4. Authentication
>>> Auth::attempt(['email' => 'user@example.com', 'password' => 'password']);

# 5. Authorization (Roles)
>>> $user = User::first();
>>> $user->hasRole('user');      # Should return true/false

# 6. Logging
>>> Log::info('Test log message');
# Check: storage/logs/laravel.log

# 7. Mail (development)
>>> Mail::raw('Test email', function ($msg) { $msg->to('test@example.com'); });
# Check: storage/logs/laravel.log
```

### SECURITY CHECKLIST

- [ ] Production `.env` has `APP_DEBUG=false`
- [ ] Production `.env` has `APP_ENV=production`
- [ ] Production database password is strong (12+ characters)
- [ ] Production `APP_KEY` is unique and kept secret
- [ ] HTTPS is enabled in production
- [ ] `.env` file is in `.gitignore`
- [ ] Sensitive logs are not committed
- [ ] Database backups are configured
- [ ] Audit logging is enabled

---

## TROUBLESHOOTING CONFIGURATION ISSUES

### COMMON CONFIGURATION PROBLEMS

| Problem | Cause | Solution |
|---------|-------|----------|
| "No APP_KEY" error | Missing encryption key | Run `php artisan key:generate` |
| "Connection refused" | Wrong DB host | Check `DB_HOST` (use `mysql` in Docker) |
| "SQLSTATE[HY000]" | Database not running | Start Docker: `docker-compose up -d` |
| Blank page in browser | Debug mode off, error not logged | Set `APP_DEBUG=true` temporarily |
| "Exceeded size" | Cache/session table full | Clear cache/sessions: `php artisan cache:clear` |
| Emails not sending | Wrong mail driver | Check `MAIL_MAILER` and credentials |
| Slow API responses | Cache not configured | Use Redis: `CACHE_STORE=redis` |
| Permission denied | Wrong file permissions | `chmod -R 755 storage bootstrap/cache` |

### CONFIGURATION DEBUGGING

**View current configuration:**

```bash
# View all configuration
php artisan config:show

# View specific configuration
php artisan config:show app
php artisan config:show database
php artisan config:show cache
```

**Test configuration:**

```bash
# Test database connection
php artisan migrate:status

# Test cache
php artisan cache:clear
php artisan cache:test

# Test queue
php artisan queue:work --tries=1

# Test mail
php artisan mail:test user@example.com

# Test logging
php artisan log:clear
php artisan tinker
>>> Log::info('Test');
```

### RESET CONFIGURATION

```bash
# Clear all caches
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear

# Rebuild cached files
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan optimize

# Fresh migration (WARNING: deletes all data)
php artisan migrate:fresh --seed
```

---

## CONFIGURATION REFERENCE SUMMARY

### KEY FILES AT A GLANCE

| File | Primary Use | Sensitive? |
|------|------------|-----------|
| `.env` | Environment variables | **YES** - Don't commit |
| `.env.example` | Template | NO - Safe to commit |
| `config/app.php` | App settings | NO |
| `config/database.php` | DB connections | References `.env` |
| `config/cache.php` | Cache config | NO |
| `config/auth.php` | Authentication | NO |
| `config/permission.php` | Authorization | NO |
| `config/audit.php` | Auditing | NO |
| `config/logging.php` | Logging | NO |
| `compose.yaml` | Docker setup | NO |
| `vite.config.js` | Build config | NO |

### ENVIRONMENT VARIABLE CATEGORIES

**Core Application (4):**
- APP_NAME, APP_ENV, APP_DEBUG, APP_URL

**Database (5):**
- DB_CONNECTION, DB_HOST, DB_PORT, DB_DATABASE, DB_USERNAME, DB_PASSWORD

**Cache (2):**
- CACHE_STORE, DB_CACHE_TABLE

**Session (2):**
- SESSION_DRIVER, SESSION_LIFETIME

**Queue (2):**
- QUEUE_CONNECTION, DB_QUEUE_TABLE

**Mail (6):**
- MAIL_MAILER, MAIL_HOST, MAIL_PORT, MAIL_USERNAME, MAIL_PASSWORD, MAIL_FROM_ADDRESS

**Authentication (3):**
- AUTH_GUARD, FORTIFY_FEATURES, JETSTREAM_STACK

**Logging (2):**
- LOG_CHANNEL, LOG_LEVEL

**Security (3):**
- APP_KEY, APP_CIPHER, SESSION_SECURE

---

## CONCLUSION

The Health Data Bank system uses a comprehensive, well-structured configuration approach that:

**Separates concerns** - Configuration files organized by function  
**Manages secrets** - Sensitive data in `.env` (not in repo)  
**Supports environments** - Different configs for dev/staging/prod  
**Enables HIPAA compliance** - Audit logging and encryption configured  
**Optimizes performance** - Caching, queue, and session configuration  
**Ensures security** - Role-based access, 2FA, CSRF protection  

Follow this guide to properly configure the Health Data Bank system for development, staging, or production deployment.

---

**Document Version:** 1.0  
**Last Updated:** 2026-03-23  
**Status:** Complete & Ready for Deployment
```
