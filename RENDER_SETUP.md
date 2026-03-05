# Render Deployment Setup Guide

## Quick Setup Steps

### 1. Create a PostgreSQL Database on Render
- Go to https://dashboard.render.com
- Click "New +" → "PostgreSQL"
- Name: `elearning-db` (or your choice)
- Click "Create Database"
- Copy the connection string from the database details page

### 2. Create a Web Service on Render
- Click "New +" → "Web Service"
- Connect your GitHub repository
- Name: `maganacademys` (or your choice)
- Runtime: `Python`
- Build command: `./build.sh`
- Start command: `gunicorn elearning.wsgi:application`
- Plan: Choose appropriate plan (free, starter, standard, etc.)

### 3. Set Environment Variables
In the Render dashboard, add these environment variables to your Web Service:

**Required:**
```
SECRET_KEY=<generate-a-secure-key>
DEBUG=False
ALLOWED_HOSTS=maganacademys.onrender.com
DATABASE_URL=<paste-database-connection-string-here>
SITE_URL=https://maganacademys.onrender.com
```

**For Cloudinary (Media Storage):**
```
CLOUDINARY_CLOUD_NAME=<your-cloudinary-cloud-name>
CLOUDINARY_API_KEY=<your-cloudinary-api-key>
CLOUDINARY_API_SECRET=<your-cloudinary-api-secret>
```

**For Email (gmail example):**
```
EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USE_TLS=True
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=<gmail-app-password>
DEFAULT_FROM_EMAIL=your-email@gmail.com
```

**For Payment Processing (optional):**
```
WAAFIPAY_MERCHANT_ID=<your-merchant-id>
WAAFIPAY_API_USER_ID=<your-api-user-id>
WAAFIPAY_API_KEY=<your-api-key>
WAAFIPAY_MODE=production
```

### 4. Deploy
- Click "Deploy" in the Render dashboard
- Monitor the deployment logs
- Once live, your app will be available at `https://maganacademys.onrender.com`

## Troubleshooting

### 500 Errors on HomePage
If you see 500 errors, check:

1. **Database Connection**
   - Verify `DATABASE_URL` is correct in environment variables
   - Ensure database is active on Render
   - Check that migrations ran successfully in deployment logs

2. **Missing Environment Variables**
   - Verify all REQUIRED variables are set
   - Check for typos in variable names
   - Ensure `CLOUDINARY_*` variables are set for media storage

3. **Check Logs**
   - Go to Render dashboard
   - Click your service
   - View deployment logs in "Logs" tab
   - Look for specific error messages

### Database Issues
If migrations failed:
1. Check the deployment logs for migration errors
2. Verify DATABASE_URL format is correct
3. Ensure database has been created on Render
4. Try redeploying (this will run migrations again)

### Static Files Issues
If CSS/JS don't load:
- Check that `collectstatic` command ran in build logs
- Verify `STATIC_ROOT` is set correctly
- Clear Render cache: Settings → Clear all build caches

## Monitoring

After deployment, monitor:
- Application logs: Render Dashboard → Logs
- Performance: Render Dashboard → Metrics
- Health checks: Render Dashboard → Health

## Scaling

The free tier includes:
- 0.5 CPU
- 512 MB RAM
- No persistent disks
- Spins down after 15 minutes of inactivity

For better performance and reliability, upgrade to a paid plan.

## Support

For issues:
1. Check Render documentation: https://render.com/docs
2. Review application logs in Render dashboard
3. Verify environment variables are correctly set
4. Ensure all required packages are in requirements.txt
