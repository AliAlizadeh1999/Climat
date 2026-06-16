# Climat
import xarray as xr
import os

# مسیر ورودی و خروجی
input_dir = r"input_dir"
output_dir = r"output_dir"

# ایجاد پوشه خروجی در صورت عدم وجود
os.makedirs(output_dir, exist_ok=True)

# محدوده مکانی مورد نظر
lat_min, lat_max = 24, 42
lon_min, lon_max = 42, 64

# گرفتن فهرست فایل‌های NetCDF
nc_files = [f for f in os.listdir(input_dir) if f.endswith('.nc')]

# حلقه روی فایل‌ها
for file in nc_files:
    in_path = os.path.join(input_dir, file)
    out_path = os.path.join(output_dir, file)

    # خواندن فایل
    ds = xr.open_dataset(in_path)

    # اصلاح طول جغرافیایی از 0–360 به -180–180 در صورت نیاز
    if ds.lon.max() > 180:
        ds = ds.assign_coords(lon=(((ds.lon + 180) % 360) - 180))
        ds = ds.sortby(ds.lon)
    
    # برش محدوده
    ds_clip = ds.sel(lat=slice(lat_min, lat_max), lon=slice(lon_min, lon_max))

    # ذخیره فایل جدید
    ds_clip.to_netcdf(out_path)

    print(f"✅ فایل {file} با موفقیت برش و ذخیره شد.")

print("🎉 تمام فایل‌ها با موفقیت پردازش شدند.")

