# ArborPI_project
from datetime import datetime, timedelta
from time import sleep
from orbit import ISS
from picamera import PiCamera
from pathlib import Path
import csv
from logzero import logger, logfile

# creating a csv file
def create_csv(data_file):
    with open(data_file, 'w') as f:
        writer = csv.writer(f)
        header = ("image", "time/date", "latitude", "longitude", "elevation")
        writer.writerow(header)

# define a function for adding the data into the csv file
def add_csv_data(data_file, data):
    with open(data_file, 'a') as f:
        writer = csv.writer(f)
        writer.writerow(data)


def convert(angle):

# Convert a skyfield Angle to an appropriate representation for EXIF files (rationals)

    sign, degrees, minutes, seconds = angle.signed_dms()
    exif_angle = f'{degrees:.0f}/1,{minutes:.0f}/1,{seconds * 10:.0f}/10'
    return sign < 0, exif_angle

# define a function for capturing an image and data in EXIF representation
def capture(camera, image):
    """Use `camera` to capture an `image` file with lat/long EXIF data."""
    point = ISS.coordinates()

    # Convert the latitude and longitude to EXIF-appropriate representations
    south, exif_latitude = convert(point.latitude)
    west, exif_longitude = convert(point.longitude)

    # Adding the coordinates and the elevation data of the target.
    camera.exif_tags['GPS.GPSLatitude'] = exif_latitude
    camera.exif_tags['GPS.GPSLatitudeRef'] = "S" if south else "N"
    camera.exif_tags['GPS.GPSLongitude'] = exif_longitude
    camera.exif_tags['GPS.GPSLongitudeRef'] = "W" if west else "E"
    camera.exif_tags['GPS.GPSElevation'] = point.elevation.km

    # Capture the image
    camera.capture(image)

# creating the base folder
base_folder = Path(__file__).parent.resolve()

# creating a logfile
logfile(base_folder/"events.log")

# setup the camera
cam = PiCamera()
cam.resolution = (1296, 972)

# creating a csv file for the data(in adition to the EXIF-file)
data_file = base_folder / 'data.csv'
create_csv(data_file)

imagecounter = 1

# record the start and the current time
start_time = datetime.now()
now_time = datetime.now()

# running a loop for 178 minutes
while (now_time < start_time + timedelta(minutes=178)):
    try:
        # get current coordinates of the overflying location
        point = ISS.coordinates()
        # store the data to the csv file
        row = (imagecounter, datetime.now(), point.latitude, point.longitude, point.elevation.km)
        add_csv_data(data_file, row)

        image_name = f"{base_folder}/photo_{imagecounter}.jpg"
        capture(cam, image_name)

        # Log info
        logger.info(f"iteration {counter}")

        imagecounter += 1
        # waiting
        sleep(15)
        # update the current time
        now_time = datetime.now()

    except Exception as e:
        logger.error(f'{e.__class__.__name__}: {e}')
