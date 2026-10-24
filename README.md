# eventhub:
A backend API for a simplified event ticketing platform, built with Django, Django ORM and Django REST Framework. Users can browse events, reserve seats and cancel reservations.

# How to run:
1.Clone the repository and open the project folder.
2.Create and activate a virtual environment:
python -m venv .venv
.venv\Scripts\activate

3.Install the dependencies:
pip install -r requirements.txt
4.Apply the migrations:
python manage.py migrate
5. Start the server:
python manage.py runserver
6.The API is available at http://127.0.0.1:8000/api/. Test it with Postman or the browsable API in your browser.

 Every request is logged in the terminal by RequestLoggingMiddleware, for example POST /api/reservations/ - 201 - 0.01s.

# Validation rules:
1.available_seats cannot exceed total_seats (events).
2.At least 1 seat must be reserved.
3.Seats can only be reserved for upcoming or ongoing events.
4.A reservation cannot ask for more seats than are available (returns 400, for example Only 1 seat(s) available.).
5. A reservation that is already cancelled cannot be cancelled again.
# Project structure:
eventhub/
  manage.py
  requirements.txt
  db.sqlite3
  eventhub/
    settings.py
    urls.py
  events/
    models.py
    serializers.py
    views.py
    urls.py
    middleware.py
    migrations/

# Postman screenshots:

successful reservation(201 created)

<img width="1366" height="638" alt="reservation-201" src="https://github.com/user-attachments/assets/f5a97042-1736-44fa-b439-5957e73def78" />


overbooking failure(400 bad request)
<img width="1352" height="488" alt="overbooking-400" src="https://github.com/user-attachments/assets/c270db21-43dc-483f-88bd-41229aec1015" />





successful cancellation 
<img width="1366" height="626" alt="cancel-200" src="https://github.com/user-attachments/assets/64fdc0a8-08fd-438d-8ed0-ff05923ef815" />





# Design decision:
I used a soft cancel: cancelling a reservation sets its status to cancelled and gives the seats back to the event, instead of deleting the reservation row. This keeps a full history of who booked and cancelled, lets the API reject a second cancel with a clear "Already cancelled" error, and means reservations_count on an event only counts confirmed reservations. Seat deduction on booking and seat restoration on cancel both happen next to the reservation write (in the serializer's create() and the cancel action), so the seat count and the reservation records change together. For production use with many concurrent users, I would wrap these in transaction.atomic() to prevent two people booking the last seat at the same time.
    
