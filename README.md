# Cyclist Power · Speed · Gear Advisor

This web application is a comprehensive tool for cyclists to simulate or track their rides, analyze performance metrics like power and speed, and receive real-time gear suggestions. It can utilize GPS for live tracking or simulate a ride with configurable parameters. Additionally, it supports connecting to a BLE (Bluetooth Low Energy) cadence sensor for more accurate data.

## Features

* **Dual Mode Operation:**
    * **Simulate Mode:** Allows users to simulate a cycling session with virtual movement.
    * **GPS Mode:** Uses the device's GPS to track actual rides, fetching live location, speed, and elevation data.
* **Comprehensive User Settings:**
    * Rider and bike mass.
    * Functional Threshold Power (FTP).
    * Coefficient of Rolling Resistance (CRR).
    * Base Coefficient of Drag Area (CdA), with an option for automatic adjustment based on gradient (climbing, flats, descents).
    * Preferred cadence range (min/max RPM).
    * Drivetrain configuration (front chainrings and rear cassette teeth).
    * Wheel circumference and crank length.
* **Live Performance Metrics:**
    * Calculated Power Output (Watts).
    * Speed (km/h).
    * Gradient (%).
    * Cadence (RPM) - from BLE sensor or estimated.
    * Detected current gear (if cadence sensor connected).
    * Suggested optimal gear based on preferred cadence and current speed.
* **BLE Cadence Sensor Integration:**
    * Connects to standard "Cycling Speed and Cadence" (CSC) BLE sensors to read live cadence data.
* **Data Logging and Export:**
    * Records detailed ride data points every 2 seconds, including timestamp, location, altitude, speed, power, cadence, gear information, gradient, CdA, and wind conditions.
    * Generates a downloadable CSV file of the ride session.
* **EmailJS Integration (Optional):**
    * Allows users to email the ride CSV data to a specified address.
    * Requires user-provided EmailJS credentials (User ID, Service ID, Template ID) for functionality.
* **Interactive Ride Map:**
    * Displays the recorded ride route on an OpenStreetMap layer using Leaflet.js.
    * Clicking on the route shows detailed metrics for that specific point in the ride.
* **External API Usage:**
    * Fetches real-time wind speed and direction from the Open-Meteo API based on current location.
    * Fetches elevation data from the Open-Meteo API when in GPS mode.
* **Ride Summary:**
    * Provides a summary upon session completion, including:
        * Average Power.
        * Average Speed.
        * Estimated Calories Burned.
        * Total Elevation Gain.

## How to Use

1.  **Open the HTML File:** Open the `index.html` (or the provided HTML file) in a modern web browser that supports Web Bluetooth (for cadence sensor) and Geolocation (for GPS mode).
2.  **Configure Settings:**
    * Fill in the form with your personal and bike details:
        * **Mode:** Choose "Simulate" or "Use GPS".
        * **Auto Pose CdA:** Check if you want CdA to adjust automatically with terrain.
        * Enter `Rider mass`, `Bike mass`, `FTP`, `CRR`, and base `CdA`.
        * Set your `Preferred cadence` range.
        * Input your `Front chainrings` and `Rear cassette` teeth numbers, separated by commas (e.g., `50,34` and `11,13,15,17,19,21,23,25,28,32`).
        * Specify `Wheel circumference` and `Crank length`.
        * Optionally, enter your `Email` if you plan to use the EmailJS feature (requires EmailJS setup).
    * The "Settings Summary" will update to show your effective CdA based on your choice.
3.  **Connect Cadence Sensor (Optional):**
    * Click the "Connect Cadence Sensor" button.
    * Your browser will prompt you to pair with a nearby BLE cadence sensor. Select your sensor.
    * The button text will change to "Cadence ✔" upon successful connection.
4.  **Start a Session:**
    * Click the "Start Session" button.
    * If in "GPS Mode", allow location access when prompted. It might take a moment to get the first GPS fix.
    * The "Live Stats" section will begin displaying real-time data.
5.  **During the Session:**
    * Monitor your power, speed, gradient, cadence, and gear suggestions.
6.  **Stop a Session:**
    * Click the "Stop Session" button.
7.  **View Summary and Export Data:**
    * After stopping, a "Ride Summary" will appear with average metrics.
    * A "Download CSV" link will be generated. Click it to save your ride data.
    * If you entered an email in settings AND have EmailJS configured (see below), the CSV might be emailed automatically, or an option to send it will appear.
8.  **View Map:**
    * After the session, a map displaying your route will pop up in a modal.
    * Click anywhere on the polyline to see detailed data for that point.
    * Click the "×" button to close the map.

## Technical Details

* **Frontend:** HTML, CSS, JavaScript.
* **Mapping:** Leaflet.js with OpenStreetMap tiles.
* **BLE Communication:** Web Bluetooth API for cadence sensor.
* **Geolocation:** Browser Geolocation API.
* **Emailing:** EmailJS Browser SDK.
* **External APIs:**
    * Open-Meteo API for current wind data (`https://api.open-meteo.com/v1/forecast`).
    * Open-Meteo API for elevation data (`https://api.open-meteo.com/v1/elevation`).
* **Key Calculations:**
    * **Power Calculation:**
        $P = (F_g + F_r + F_a) \times v$
        Where:
        * $P$ = Power
        * $F_g = m \cdot g \cdot \sin(\theta)$ (Gravitational force component)
        * $F_r = m \cdot g \cdot C_{rr} \cdot \cos(\theta)$ (Rolling resistance force)
        * $F_a = 0.5 \cdot \rho \cdot C_d A \cdot v_{app}^2$ (Aerodynamic drag force)
        * $m$ = total mass (rider + bike)
        * $g$ = acceleration due to gravity (9.80665 m/s²)
        * $\theta$ = road gradient angle
        * $C_{rr}$ = Coefficient of Rolling Resistance
        * $\rho$ = air density (1.225 kg/m³)
        * $C_d A$ = Coefficient of Drag Area (potentially adjusted for pose)
        * $v_{app}$ = apparent wind speed (cyclist speed + relative wind speed)
        * $v$ = cyclist speed
    * **Gear Suggestion:** Aims to find a gear that places the cyclist's cadence within their preferred range at the current wheel RPM.
    * **Distance:** Haversine formula for calculating distance between GPS coordinates.

## Configuration for EmailJS

To enable the feature of emailing the CSV data directly after a session or through the "Send CSV" button:

1.  **Sign up for EmailJS:** If you don't have an account, create one at [dashboard.emailjs.com](https://dashboard.emailjs.com/).
2.  **Add a New Service:** Connect your email provider (e.g., Gmail, Outlook) as a service in EmailJS. Note the **Service ID**.
3.  **Create an Email Template:**
    * Design an email template in EmailJS.
    * Your template should accept the following parameters:
        * `{{to_email}}`: The recipient's email address.
        * `{{ride_filename}}`: The name of the CSV file.
        * `{{ride_data}}`: The CSV content (as a string). You'll likely need to configure this as an attachment or include it in the body. For attachments, EmailJS typically expects a base64 string or a direct content string depending on your setup. The current script sends `csvContent` directly.
    * Note the **Template ID**.
4.  **Get your User ID:** Find your **User ID** under the "Account" section in EmailJS.
5.  **Enter Credentials in the App:**
    * When the "Enter email to send CSV" section appears after a ride (if no initial email was provided or automatic sending failed), you can input your:
        * **User ID**
        * **Service ID**
        * **Template ID**
    * These fields are password-type for basic privacy, but the credentials are used directly in the client-side JavaScript.
    * Alternatively, for a more permanent solution, you can hardcode your `EMAILJS_USER_ID`, `EMAILJS_SERVICE`, and `EMAILJS_TEMPLATE` variables directly in the `<script>` section of the HTML file (lines 131-133).
        ```javascript
        // const g=9.80665, rho=1.225, efficiency=0.23;
        // let EMAILJS_USER_ID='',EMAILJS_SERVICE='',EMAILJS_TEMPLATE='';
        // // ---- becomes ----
        // const g=9.80665, rho=1.225, efficiency=0.23;
        // let EMAILJS_USER_ID='YOUR_USER_ID',EMAILJS_SERVICE='YOUR_SERVICE_ID',EMAILJS_TEMPLATE='YOUR_TEMPLATE_ID';
        ```
    * **Note:** Storing API keys or sensitive credentials client-side is generally not recommended for public-facing applications due to security risks. For personal use, this might be acceptable.

## Disclaimer

* Calculations are estimates and depend on the accuracy of user inputs and external data sources.
* GPS accuracy can vary.
* Ensure you have the necessary permissions and comply with the terms of service for any APIs used (e.g., Open-Meteo).
