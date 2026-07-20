# Development Challenge

## Part 1 – A humble beginning

### Context

The FAA (Future Assets Analytics) team at Axpo is responsible for analysing the feasibility of innovative projects to build the future generation of assets.

In this case, the team is studying whether to build a new Wind Farm in Antarctica. For that, they need historical weather data from the AEMET (Spanish Meteorological Agency).

The AEMET provides a free-licensed REST API to allow the dissemination and reuse of the Agency's meteorological and climatological information. The documentation of this API can be found at: <https://opendata.aemet.es/centrodedescargas/inicio>.

The team has requested you to create a Python API that they can use to retrieve easily the required AEMET data.

### Requirements

The objective of the exercise is to develop a web API which meets the requirements below.

1. You shall create a web service in **FastAPI** or **Flask**.

2. The plugin shall use the endpoint:

   ```
   /api/antartida/datos/fechaini/{fechaIniStr}/fechafin/{fechaFinStr}/estacion/{identificacion}
   ```

3. This endpoint is providing the timeseries of different measurements in the specified meteo station and in the specified time range.

4. The API key required by the source API shall be an environment variable. It can be obtained from the previously commented URL. Please use a personal e-mail account, as we observed that corporate accounts were blocked.

5. **Parameters:** The user shall be able to specify the following parameters:

   a. **Datetime Start** with format `AAAA-MM-DDTHH:MM:SS`.

   b. **Datetime End** with format `AAAA-MM-DDTHH:MM:SS`.

   c. **Location** (optional) should indicate the location of the datetimes you passed (e.g., `Europe/Berlin`). Alternatively, you could also ask for the time zone shift in the datetimes (e.g., `+02:00`).

   d. **Meteo Measurement Station:** From the API doc there shall only be two possible selections:
      - “Meteo Station Gabriel de Castilla”
      - “Meteo Station Juan Carlos I”

   e. **Time Aggregation:** Possible values shall be `None`, `Hourly`, `Daily` and `Monthly`. Be aware that a daily and monthly aggregation requires considering the time zone location of the meteo station.

   f. A list of the required **data types**: temperature, pressure, speed.
      - User can select zero to three types (see table below); if zero, return all.

6. **Granularity:** The granularity of the data from the API is in 10 min.

   a. The output dataset shall be aggregated based on the user selection of the Time Aggregation field.

   b. The output dataset shall not be aggregated when `None` is selected.

7. Below a table with the main attributes to be considered from the source API:

   | API Response Name | Dataset Field Name   | Description                        |
   |-------------------|----------------------|------------------------------------|
   | `nombre`          | Station              | Name of the station                |
   | `fhora`           | Datetime             | Date and time of the measurement   |
   | `temp`            | Temperature (°C)     | Temperature in °Celsius            |
   | `pres`            | Pressure (hPa)       | Pressure in hPa                    |
   | `vel`             | Speed (m/s)          | Wind speed in m/s                  |

8. The Datetime field of the output dataset shall be in CET/CEST time zone (location `Europe/Madrid`). Nevertheless, it needs to include the time zone offset (e.g., `+02:00`).

9. Confirm the behaviour of your API during Daylight-Saving Time (DST).

10. You should provide appropriate tests in a proper way, even if they do not cover every possible scenario.

11. The code shall be available in a Git repository.

---

## Part 2 – Traders learned about this service…

### Context

Traders have discovered your service and are eager to expand its capabilities to include all available meteorological stations. This expansion will result in thousands of requests from internal applications, making it crucial to avoid overloading the source API. Since the API adds new data only a few times per day, consumers will frequently request updates to enhance their forecasts. This, in turn, supports traders in operating efficiently in the intraday market, making this service highly critical and requiring high availability during business hours.

### Requirements

1. Remove the restriction of the meteo stations from Part 1 (allow multiple meteo stations).

2. Use SQLite to store the data you read from the source API to avoid overloading it; pay attention to best practices of DB manipulation.

3. Include proper logging for troubleshooting.

---

## Part 3 – Front-end for convenience

### Context

Although this is primarily a backend service, sometimes users want a convenient way to quickly view the data.

### Requirements

1. Create a front-end service where the user could query the data in a table, maybe a chart.

2. You might need to create supporting endpoints for this.

3. There is no restriction — just checking the concepts, code organization and use of frontend techniques.

---

## Important Tips

The engineer assessing this challenge will focus on:

- Proper use and knowledge of common Python packages
- Potential scalability
- Testing practices
- Code maintainability, readability, and clarity
- Ease of refactoring and improving the code, even with errors
- Clear README instructions for running the code
- Usability of the front-end (bonus points for user-friendly design)

If you couldn’t implement an element because of time, please at least point it out in the README or in TODO comments.
