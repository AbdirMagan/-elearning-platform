# Render Deployment - 500 Error Fix

## What Was Wrong

Your deployment was returning 500 errors on the homepage because of multiple configuration issues:

### Issues Found and Fixed:

#### 1. **ASGI/WSGI Mismatch** ✅
- **Problem**: `render.yaml` was configured to use ASGI (`elearning.asgi:application`) with uvicorn workers, but Django settings use WSGI
- **Fix**: Updated `render.yaml` to use correct WSGI command: `gunicorn elearning.wsgi:application`
- **File**: `render.yaml`

#### 2. **Domain Mismatch** ✅
- **Problem**: ALLOWED_HOSTS had `maganacademy.onrender.com` but actual domain is `maganacademys.onrender.com` (with 's')
- **Fix**: Updated `ALLOWED_HOSTS` default to include the correct domain
- **File**: `elearning/settings.py` line 19

#### 3. **Missing Cloudinary Configuration** ✅
- **Problem**: Cloudinary environment variables were required without defaults, causing failures if not set
- **Fix**: Added default empty strings to allow the app to start even without Cloudinary credentials (for testing)
- **File**: `elearning/settings.py` lines 113-116

#### 4. **Build Process Issues** ✅
- **Problem**: `build.sh` didn't have proper error messages or logging
- **Fix**: Added echo statements and `--noinput` flag to migrations
- **File**: `build.sh`

## Changes Made

### 1. render.yaml
```yaml
# BEFORE:
startCommand: python -m gunicorn elearning.asgi:application -k uvicorn.workers.UvicornWorker

# AFTER:
startCommand: gunicorn elearning.wsgi:application
```

### 2. settings.py - ALLOWED_HOSTS
```python
# BEFORE:
ALLOWED_HOSTS = config('ALLOWED_HOSTS', default='localhost,127.0.0.1,maganacademy.onrender.com', cast=Csv())

# AFTER:
ALLOWED_HOSTS = config('ALLOWED_HOSTS', default='localhost,127.0.0.1,maganacademys.onrender.com', cast=Csv())
```

### 3. settings.py - Cloudinary Configuration
```python
# BEFORE:
CLOUDINARY_STORAGE = {
    'CLOUD_NAME': config('CLOUDINARY_CLOUD_NAME'),
    'API_KEY': config('CLOUDINARY_API_KEY'),
    'API_SECRET': config('CLOUDINARY_API_SECRET'),
}

# AFTER:
CLOUDINARY_STORAGE = {
    'CLOUD_NAME': config('CLOUDINARY_CLOUD_NAME', default=''),
    'API_KEY': config('CLOUDINARY_API_KEY', default=''),
    'API_SECRET': config('CLOUDINARY_API_SECRET', default=''),
}
```

### 4. build.sh
Added logging and improved error handling for better debugging

## Next Steps

1. **Redeploy on Render**:
   ```
   git add .
   git commit -m "Fix deployment configuration for Render"
   git push
   ```
   This will automatically trigger a redeploy on Render

2. **Verify Environment Variables**:
   Go to Render Dashboard → Your Service → Environment
   
   **At minimum, set these:**
   - ✅ `SECRET_KEY` (generate a random secure key)
   - ✅ `DATABASE_URL` (your Neon PostgreSQL connection string)
   - ✅ `DEBUG=False`
   - ✅ `ALLOWED_HOSTS=maganacademys.onrender.com`

   **Recommended (for media storage):**
   - `CLOUDINARY_CLOUD_NAME`
   - `CLOUDINARY_API_KEY`
   - `CLOUDINARY_API_SECRET`

3. **Monitor Deployment**:
   - Go to Render Dashboard
   - Click on your service `maganacademys`
   - Watch the "Deploy log" for any errors
   - Once deployed, test: https://maganacademys.onrender.com

## If 500 Errors Still Occur

1. **Check the logs**:
   - Render Dashboard → Your Service → Logs
   - Look for error messages in the deployment or runtime logs

2. **Common causes**:
   - Missing `DATABASE_URL` environment variable
   - Database not created or not reachable
   - Required environment variable not set

3. **Enable debug mode temporarily**:
   - Set `DEBUG=True` in environment variables
   - This will show detailed error pages
   - **Remember to set back to False when resolved!**

4. **Manually run migrations**:
   If migrations didn't run during deployment:
   - This should happen automatically, but if needed:
   - Go to Render → Your Service → Shell
   - Run: `python manage.py migrate --noinput`

## Files Created/Updated

- ✅ `render.yaml` - Fixed startCommand
- ✅ `elearning/settings.py` - Fixed ALLOWED_HOSTS and Cloudinary defaults
- ✅ `build.sh` - Added logging and improvements
- ✅ `.env.example` - Updated with all required variables
- ✅ `RENDER_SETUP.md` - Complete Render setup guide (NEW)
- ✅ `RENDER_DEPLOYMENT_FIX.md` - This troubleshooting guide (NEW)

## Additional Resources

- Render Documentation: https://render.com/docs
- Django Deployment Guide: https://docs.djangoproject.com/en/stable/howto/deployment/
- Cloudinary Django Integration: https://cloudinary.com/documentation/django_integration
