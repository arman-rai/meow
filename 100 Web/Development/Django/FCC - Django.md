### **Django Crash Course: Python Web Framework Overview**

This crash course provides a foundational understanding of the Django web framework, guiding beginners through environment setup, core concepts (models, views, templates), and the development of dynamic web applications. The course emphasizes practical application, culminating in building a full Django application.

#### **I. Core Django Concepts and Architecture**

- **Web Framework Definition:** A web framework provides a "structured way of building web applications." It defines a structure for writing code, unlike a library, which is a collection of functionalities for specific tasks.
- **MVT (Model-View-Template) Architecture:** Django follows the MVT architectural pattern, which divides application responsibilities into three distinct parts:
- **Model:** "Responsible for processing data. It represents how our data is structured, stored, and retrieved." Essentially, it handles the database interactions.
- **View:** "Responsible for handling the user interactions with our application... where we take user requests, process them, and send back responses." This deals with the application's logic.
- **Template:** "Deals with the user interface or what the user actually sees." This includes HTML markup with placeholders for dynamic content.

#### **II. Environment Setup and Project Structure**

- **Prerequisites:Python:** The latest version should be downloaded.
- **VS Code:** Recommended as the code editor, with the Python and SQL Lite Viewer extensions installed.
- **Virtual Environments with pipenv:**pipenv is used to create a "virtual environment that is going to store all the versions of the different packages we're going to use with our application."
- **Installation:** pip install pipenv
- **Activation:** pipenv shell
- **Django Installation:** Once pipenv is activated, Django is installed within the virtual environment: pipenv install Django.
- **Project and App Creation:Project:** A Django project is created using django-admin startproject <project_name> (e.g., django-admin startproject firstproject). This creates a folder containing project configurations.
- **Apps:** "Our Django project can have multiple apps. An app in Django is a collection of functionalities that focus on a particular aspect of our overall project." Examples include authentication, blog posts, or messaging. Apps are created within a project directory using python manage.py startapp <app_name> (e.g., python manage.py startapp firstapp).
- **manage.py:** This script is central to Django development. "Once we've created our project we can use Django admin and manage.py interchangeably but throughout this course we're going to be using manage.py." Commands are executed as python manage.py <command>.
- **Interpreter Configuration:** In VS Code, it's crucial to "change our interpreter to the one we set up in the virtual environment" (the one ending in pipenv).
- **Adding Apps to Project Settings:** For an app to function within a project, its name must be added to the INSTALLED_APPS list in the project's settings.py file.

#### **III. Views and URL Mapping**

- **Views:** Views handle requests and return responses. They can be created in two ways in Django:
- **Function-based views:** Defined as Python functions that take request as a parameter and return an HttpResponse object. Example: def hello_world(request): return HttpResponse("hello world")
- **Class-based views:** Defined as Python classes that inherit from django.views.View. They typically implement HTTP method functions like get() that take self and request as parameters and return an HttpResponse. Example: class HelloEthiopia(View): def get(self, request): return HttpResponse("hello Ethiopia")
- **request and response Objects:** These objects are fundamental for user-application communication.
- request provides information about the user's request, including request.method (GET, POST, etc.), request.GET and request.POST for parameters, request.cookies, and request.files.
- response is "what we send back to the user."
- **URL Mapping (Routing):** The process of "linking the view functions and classes to addresses."
- **App-level urls.py:** A urls.py file is created within each app to define paths for that app's views. It uses path from django.urls and typically defines a urlpatterns list.
- For function-based views: path('address/', views.function_name)
- For class-based views: path('address/', views.ClassName.as_view()) (requires .as_view() method).
- **Project-level urls.py:** The project's urls.py file uses the include function (also from django.urls) to incorporate the URL patterns from individual apps. Example: path('app/', include('firstapp.urls')). This creates a hierarchical URL structure.

#### **IV. Models and Database Interaction**

- **Model Definition:** Models are Python classes that inherit from models.Model and define the structure of data to be stored in the database. Each attribute in the class corresponds to a column in a database table.
- **Field Types:** Django provides various field types, such as models.CharField(max_length=255), models.IntegerField(), models.DateField(auto_now=True), etc.
- **Migrations:** "A migration is a process that converts whatever changes we make in the model.py file to our actual database."
- **Creating Migrations:** python manage.py make migrations
- **Applying Migrations:** python manage.py migrate
- **Database Backend:**Django "by default uses SQLite database, which is a lightweight serverless database."
- Django also supports other databases like MySQL. To use MySQL, a driver (mysqlclient) must be installed, and the DATABASES configuration in settings.py must be updated with the MySQL engine, database name, user, password, host, and port.
- **Admin Panel Interaction:**Models can be registered with the Django admin site by adding admin.site.register(YourModel) in the app's admin.py file. This allows for easy data management through the admin interface.
- **__str__ method:** Defining a __str__ method in a model class (e.g., def __str__(self): return self.name) provides a more descriptive representation of model objects in the admin panel instead of "Menu object (1)".

#### **V. Forms**

- **Purpose:** Forms are used to "take input from the user." Django simplifies combining forms and models to store user data.
- **forms.py:** A file created within an app to define forms.
- **ModelForm:** For forms directly linked to a model, ModelForm is used. This class inherits from forms.ModelForm.
- **Meta class:** Within the form class, a Meta class is defined to link the form to a specific model (model = YourModel) and specify which fields to include (fields = '__all__' for all fields, or a list of specific fields).
- **CSRF Token:** When submitting forms with the POST method, a Cross-Site Request Forgery (CSRF) token is required for security. It's added to templates using {% csrf_token %}.
- **Handling Form Submission in Views:**In a function-based view, an instance of the form is created.
- The request.method is checked to see if it's a POST request (meaning the form was submitted).
- If it's a POST request, the form is re-instantiated with request.POST data: form = ReservationForm(request.POST).
- form.is_valid() is called to validate the submitted data.
- If valid, form.save() is called to save the data to the associated model in the database.

#### **VI. Templates and Static Files**

- **Template Folder:** HTML files for the user interface are typically stored in a templates folder within an app.
- **Template Inheritance:** Django templates support inheritance, reducing code repetition.
- A base.html file acts as a "blueprint," containing common elements (header, footer, etc.) and defining {% block content %} and {% endblock %} tags.
- Other HTML files (e.g., menu.html) "extend" the base template using {% extends 'base.html' %} and then define their specific content within {% block content %}.
- **Displaying Dynamic Data:**Variables from views are passed to templates as a dictionary in the render function (e.g., render(request, 'index.html', {'form': form_variable})).
- Variables are displayed in templates using double curly braces: {{ variable.attribute }}.
- **Control Flow (Loops and Conditionals):** Templates support basic control flow using {% for item in list %}... {% endfor %} for loops and {% if condition %}... {% endif %} for conditionals.
- **load static:** {% load static %} is used to load static files like images, although not fully covered in this excerpt.

#### **VII. Admin Panel**

- **Purpose:** The Django admin panel is an "interface that has many functionalities related to our application," primarily for managing data (models, users, groups).
- **Creating a Superuser:** To access the admin panel, a superuser account is required. This is created using python manage.py createsuperuser.
- **User and Group Management:** The admin panel allows for creating new users, assigning passwords, and managing permissions by adding users to groups or directly assigning permissions.
- **Populating Models:** Registered models appear in the admin panel, allowing administrators to easily add, edit, and delete data directly from the browser interface.

#### **VIII. Building a Sample Application (Little Lemon Restaurant)**

The course concludes with building a menu page and individual menu item pages for a restaurant called "Little Lemon."

- **Menu Model:** A Menu model is created with name (CharField) and price (IntegerField), and later a description (CharField).
- **Admin Registration:** The Menu model is registered to the admin site.
- **Views:**A menu view fetches all menu items using Menu.objects.all() and renders them to menu.html.
- A display_menu_item view takes a pk (primary key) parameter from the URL to retrieve a specific menu item using Menu.objects.get(pk=pk) and renders it to menu_item.html.
- **Templates:**menu.html extends base.html and uses a for loop to display each menu item's name and price. Each item's name is wrapped in an anchor tag with an href linking to its individual page using {% url 'menu_item' pk=item.pk %}.
- menu_item.html also extends base.html and displays the name, description, and price of a specific menu item.
- **URL Mapping:** URL patterns are defined for both the main menu page and individual menu item pages, including capturing the primary key for individual item access: path('menu_item/<int:pk>/', views.display_menu_item, name='menu_item').