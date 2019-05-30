
1. Log in our administration.

    ```
    https://api.mikaponics.com/admin/
    ```

2. Go to the **Device** section.

3. Click **Add Device** button.

4. Fill out the form. Here is an example selection:

    ```
    User: <Select a user>
    Product: Mikapod
    Data Interval (Minutes): 1
    Timezone: America/Toronto
    Hardware Manufacturer: Raspberry Pi Foundation
    Hardware Product Name: Raspberry Pi 3 Model B+
    Hardware Product ID: PI3P
    Hardware Product Serial: 538319
    Order: <Select any orders>
    ```

5. Click **Add Instrument** button.

6. Fill out the form. Here is an example selection for the **Humidity** instrument:

    ```
    Device: <Select a device>
    Type of: Humidity
    Configuration: {"serial_number": 538319, "channel_number": 0, "hub_port_number": 0}
    Hardware Manufacturer: Phidgets Inc.
    Hardware Product Name: Humidity Phidget
    Hardware Product ID: HUM1000_0
    Hardware Product Serial: 538319
    Max value: 100

    # ...

    Min value: 0

    # ...

    ```

7. Fill out the form. Here is an example selection for the **Temperature** instrument:

    ```
    Device: <Select a device>
    Type of: Temperature
    Configuration: {"serial_number": 538319, "channel_number": 0, "hub_port_number": 0}
    Hardware Manufacturer: Phidgets Inc.
    Hardware Product Name: Humidity Phidget
    Hardware Product ID: HUM1000_0
    Hardware Product Serial: 538319
    Max value: 85

    # ...

    Min value: -40

    # ...

    ```

8. Go back to the **User** details and set the **Was Onboarded** to be **True**.
