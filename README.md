# ArborPI_project
from datetime import datetime, timedelta
from time import sleep
from orbit import ISS
from picamera import PiCamera
from pathlib import Path
import csv

def create_csv(data_file):
    with open(data_file, 'w') as f:
        writer = csv.writer(f)
        header = ("image", "time/time", "latitude", "longitude", "elevation")
        writer.writerow(header)
        
def add_csv_data(data_file, data):
        with open(data_file, 'a') as f:
            writer = csv.writer(f)
            writer.writerow(data)

def convert(angle):
    """
    Convert a `skyfield` Angle to an EXIF-appropriate
    representation (rationals)
    e.g. 98° 34' 58.7 to "98/1,34/1,587/10"

    Return a tuple containing a boolean and the converted angle,
    with the boolean indicating if the angle is negative.
    """
    sign, degrees, minutes, seconds = angle.signed_dms()
    exif_angle = f'{degrees:.0f}/1,{minutes:.0f}/1,{seconds * 10:.0f}/10'
    return sign < 0, exif_angle

def capture(camera, image):

    """Use `camera` to capture an `image` file with lat/long EXIF data."""
    point = ISS.coordinates()

    # Convert the latitude and longitude to EXIF-appropriate representations
    south, exif_latitude = convert(point.latitude)
    west, exif_longitude = convert(point.longitude)

    # Set the EXIF tags specifying the current location
    camera.exif_tags['GPS.GPSLatitude'] = exif_latitude
    camera.exif_tags['GPS.GPSLatitudeRef'] = "S" if south else "N"
    camera.exif_tags['GPS.GPSLongitude'] = exif_longitude
    camera.exif_tags['GPS.GPSLongitudeRef'] = "W" if west else "E"
    camera.exif_tags['GPS.GPSElevation'] = point.elevation.km

    # Capture the image
    camera.capture(image)

base_folder = Path(__file__).parent.resolve()

cam = PiCamera()
cam.resolution = (1296, 972)

data_file = base_folder/'data.csv'
create_csv(data_file)

imagecounter = 1

start_time = datetime.now()
now_time = datetime.now()

while (now_time < start_time + timedelta(minutes=177)):

    point = ISS.coordinates()
    
    row = (imagecounter, datetime.now(), point.latitude, point.longitude, point.elevation.km)
    add_csv_data(data_file, row)-
    
    image_name = f"{base_folder}/gps{imagecounter}.jpg"
    capture(cam, image_name)
    
    imagecounter += 1
 
    sleep(30)
    # Update the current time
    now_time = datetime.now()
    
