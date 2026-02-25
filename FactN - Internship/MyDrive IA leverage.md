## AI leverage in MyDrive

**Primary friction points and AI mitigation strategies**

| Customer pain point              | Manifestation in workflow                                | MyDrive AI/ML solution                                                     |
| -------------------------------- | -------------------------------------------------------- | -------------------------------------------------------------------------- |
| Lack of transparency in repairs. | Hidden fees and opaque repair estimates.                 | Computer vision-based damage assessment and standardized AI pricing logic. |
| Trust deficit                    | Skepticism regarding the necessity of suggested repairs. | AI-generated health reports with visual evidence and failure predictions.  |

**1. Enhancing Garage Services through Diagnostic AI**
The main advantage of AI leveraging in MyDrive is to shift consumer behavior from reactive "break-fix" cycles to proactive maintenance (or Predictive Maintenance - PdM) by analyzing vehicle usage patterns and sensor data, thereby extending vehicle lifespan, reducing long-term costs.

Related features,
- AI-powered diagnostics & vehicle inspection
	By analyzing fault codes, repair history, technician notes. 
- Smart inventory & part forecasting
	Predict what you will need based on past jobs and active repair orders. 
	Ex: Detect brake pads are running low -> find the best/nearest vendors -> suggest ordering the parts from the vendor
- Predictive maintenance & Scheduling
	By analyzing sensor data and service records. 
	Suggest maintenance and help the user schedule a maintenance appointments early.

Need a mechanism to read vehicle sensor data, enter maintenance information.

Ex: OBD2 port 
Some OBD2 devices comes with Bluetooth.

OBDs Provide data like,
- Engine RPM
- Vehicle speed
- Engine coolant temperature
- Throttle position
- Intake Air Temperature (IAT) : Temperature of air entering the engine
- Mass Air Flow : (MAF)
- Manifold Absolute Pressure(MAP) : Pressure in the intake manifold
- Fuel system status
- Fuel trims : Percentage adjustments for fuel mixture (useful for diagnosing vacuum leaks or sensor failures)
- Oxygen sensor voltage : monitor exhaust oxygen levels
- Engine load : Percentage of maximum engine capacity being used
- Fuel Rail Pressure : Fuel pressure in the injection system
- Spark ignition timing
- Battery voltage
Diagnostic Trouble Codes(DTCs) & System status,
- check Engine Light(MIL) Status : Indicates if the Malfunction Indicator Lamp is on or off
- Diagnostic Trouble Codes (DTCs) : Retrieves codes related to powertrain (P), body (B), chassis (C), and network (U) errors
- Freeze Frame Data : A snapshot of all sensor data at the exact moment a fault code was triggered (useful for diagnosing intermittent issues)
- Emission Readiness Monitors : Shows if emissions systems are ready for inspection.
- Distance/Time with MIL ON : Total mileage or runtime with the check engine light on.
Vehicle Information,
- VIN : Vehicle Identification  Number
- Calibration ID : ECU calibration numbers
- Ignition counters : Number of ignition cycles
Some advanced OBDs provides,
- Hybrid/Electric Battery Info : State of Charge (SoC), battery temperature, and voltage in EVs.
- ABS/Airbag Data : manufacturer-specific ABS (braking) and SRS (airbag) data.
- Transmission Temp : Temperature of transmission fluid.
- Fuel level Input : Level of fuel in the tank.

Prediction we can do using an OBD2 data using Machine learning,
- Predict maintenance before failure happens
	

**2. Intelligent dispatching and real-time coordination**
MyDrive can implement AI agent that utilize Natural Language Processing(NLP) to handle **"initial service request"** via voice or text. These agents can instantly assess the urgency of a situation/severity of the situation and prioritize them accordingly. 

By integrating with real-time GPS data and traffic APIs, the platform can match the nearest available provider with the appropriate equipment for the job. 
https://www.voiceflow.com/ai/towing-companies
https://towrankers.com/

**3. Dynamic pricing and Transparent Quoting**
AI-driven dynamic pricing models can analyze multiple variables simultaneously(distance, vehicle type, current demand spikes, weather condition,...) to calculate the optimal price to balance provider profitability with consumer willingness to pay. 
https://www.youngurbanproject.com/dynamic-pricing-algorithms/
https://competera.ai/resources/articles/dynamic-pricing-algorithm
https://coralogix.com/ai-blog/dynamic-pricing-models-types-algorithms-and-best-practices/
This provides an "Uber-like" experience where the price is guaranteed upfront, eliminating the consumer fear of overpaying during an emergency.

**4. AI and ML in the Spare parts marketplace**
Integrate a AI-based spare part detector for ensuring a part actually fits  the specific year, make, model of a vehicle. 
Ex: 
	A customer holding a broken bracket and MyDrive's computer vision algorithm then analyze the contours, labels, and design elements to identify the part and find compatible replacements.

Other related features, 
- Semantic search - Many customers do not know the technical name of a part. AI models trained on automotive terminology allows users to search using natural descriptions(ex: "front passenger door clicker") and map them to the correct component(ex: "door lock actuator")

**5. Enhance Platform Administration with AI**
- Automated reviews & feedbacks
	Read customer reviews which show patterns like common complaints or praise. Apply an reward-punishment mechanism based on these patterns for service providers or flag them to admins to take proper action. This helps to reduce operational effort for admins and enhance the user-experience. 
- Fraud-detection/Content moderation
	Reduce the operational cost/effort of "Review moderators" and agents responsible for resolving disputes. 
	Natural Language processing(NLP) models can evaluate thousands of customer reviews and identify the emotional tones and authenticity of the feedback to detect "review bombing" and remove them.

6. **Automated crash detection and user guiding via AI with voice assistance.**
IoT solution can be designed using,
- accelerometers(detect rapid acceleration changes, G-force changes),
- Gyroscope(detect if the vehicle has tilted or rolled over), 
- Vibration sensor(detect high-intensity shocks),
- pressure/force sensors(measure the impact force upon collision), 
- GPS(location tracking), and 
- GSM(communicate over the mobile networks) 
- Camera(capture real-time image for crash detection) modules to detect crashes. 
MyDrive can execute the **"automatic accident assistant"** service which communicates with the vehicle owner using voice commands/real-time conversations. If the severity of the accident is high, MyDrive can directly send an alert so the the moderators/agents can act accordingly.