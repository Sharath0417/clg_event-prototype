"Campus Event Reporting Prototype"
 requirements 
 fastapi==0.115.0
uvicorn==0.30.6
SQLAlchemy==2.0.36
pydantic==2.9.2
pydantic-settings==2.5.2
python-dotenv==1.0.1
qrcode==7.4.2
pillow==10.4.0
httpx==0.27.2
pytest==8.3.3
psycopg2-binary==2.9.9

git repo 
git clone https://github.com/sharath04/campus-event-platform.git
cd campus-event-platform/backend


Download the main.py
 run cmds 
 
 python -m venv venv
source venv/bin/activate   # Linux / Mac
venv\Scripts\activate      # Windows
  used PostgreeSQl 
  postgresql://username:Patil@2004@localhost:5432/campus_events
  run cmds 
  uvicorn app.main:app --reload
uvicorn app.main:app --reload
   


   Features / Functionality

List the main things your project does:

Admins can create, update, and delete events.

Students can browse events, register, and provide feedback.

Track attendance and popularity of events.
tools  = FastAPI (backend), PostgreSQL (database), Python, HTML/CSS/


📝 Design Document
Here is a design document addressing the data, schema, APIs, workflows, and edge cases for the Campus Event Management Platform.
________________________________________
1. Data to Track
The system needs to track key information related to events and student interactions.
•	Event Creation:
o	Event ID (unique identifier).
o	Event Name, Description, Date, Time, and Location.
o	Event Type (e.g., Hackathon, Workshop, Fest, Tech Talk).
o	Admin ID (creator's reference).
•	Student Registration:
o	Registration ID (unique identifier).
o	Student ID.
o	Event ID.
o	Registration Timestamp.
•	Attendance:
o	Attendance ID (unique identifier).
o	Student ID.
o	Event ID.
o	Check-in Timestamp.
•	Feedback:
o	Feedback ID (unique identifier).
o	Student ID.
o	Event ID.
o	Rating (1-5).
o	Comments (optional).
________________________________________
2. Database Schema
The database will follow a relational schema to manage the relationships between different entities. A single large dataset is assumed to simplify reporting across all colleges, with a college_id column to handle data separation.
ER Diagram Sketch:
Table Schemas:
•	colleges: college_id (PK), name
•	admins: admin_id (PK), name, email, college_id (FK)
•	students: student_id (PK), name, email, college_id (FK)
•	events: event_id (PK), name, description, date, time, location, type, admin_id (FK), college_id (FK)
•	registrations: registration_id (PK), event_id (FK), student_id (FK), timestamp
•	attendance: attendance_id (PK), event_id (FK), student_id (FK), timestamp
•	feedback: feedback_id (PK), event_id (FK), student_id (FK), rating, comments
________________________________________
3. API Design
The API will use RESTful principles to provide a clear and structured interface. All endpoints will be prefixed with /api/v1/.
Action	Endpoint	Method	Request Body (JSON)	Response (JSON)	Description
Create Event	/events	POST	{ "name": "...", "description": "...", "date": "...", "admin_id": "..." }	{ "event_id": "..." }	Admin creates a new event.
Register Student	/events/{event_id}/register	POST	{ "student_id": "..." }	{ "registration_id": "..." }	Student registers for an event.
Mark Attendance	/events/{event_id}/attend	POST	{ "student_id": "..." }	{ "status": "success" }	Admin marks a student as attended.
Submit Feedback	/events/{event_id}/feedback	POST	{ "student_id": "...", "rating": 4, "comments": "..." }	{ "status": "success" }	Student submits feedback on an event.
Generate Report	/reports/{report_type}	GET	(Query params for filters)	{ "data": [...] }	Retrieves various reports. report_type can be popularity, participation, etc.
Export to Sheets
________________________________________
4. Workflows
Registration Workflow:
1.	Student browses events on the app.
2.	Student selects an event and taps "Register."
3.	Student App sends a POST request to /api/v1/events/{event_id}/register with the student's ID.
4.	API validates the request and checks if the student is already registered.
5.	API creates a new record in the registrations table.
6.	API responds with a success message and a registration_id.
Attendance Workflow:
1.	Admin uses the portal to manage an event.
2.	Admin checks in a student (e.g., by scanning a QR code or manual entry).
3.	Admin Portal sends a POST request to /api/v1/events/{event_id}/attend with the student's ID.
4.	API validates the request and checks if the student is registered for the event.
5.	API creates a new record in the attendance table.
6.	API responds with a success message.
Reporting Workflow:
1.	Admin requests a report from the Admin Portal.
2.	Admin Portal sends a GET request to /api/v1/reports/popularity.
3.	API executes a database query (e.g., SELECT event_id, COUNT(*) FROM registrations GROUP BY event_id ORDER BY COUNT(*) DESC).
4.	API formats the query results into a JSON response.
5.	API sends the report data back to the Admin Portal.
________________________________________
5. Assumptions & Edge Cases
•	Duplicate Registrations: The system will prevent a student from registering for the same event more than once. The API's registration endpoint will check for an existing registrations record for the given event_id and student_id and return an error if found.
•	Missing Feedback: Feedback is optional. Reports will handle NULL values gracefully, for instance, by calculating the average score based only on the available feedback records.
•	Cancelled Events: If an event is cancelled, a flag (e.g., is_cancelled boolean) will be set on the events table. The student app will not display cancelled events, and registration attempts will be rejected.
•	Event ID Uniqueness: Event IDs will be unique across all colleges, with the college_id serving as a filter to provide context. This simplifies data management and cross-college reporting.
•	Scale: The design assumes that a single database instance with proper indexing will be sufficient for the initial scale of ~50 colleges and ~20 events per semester. Future scaling might require sharding or other database optimization techniques.

 
      

