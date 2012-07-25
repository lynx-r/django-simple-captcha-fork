Using django-simple-captcha
===========================

Installation
+++++++++++++

1. Download ``django-simple-captcha`` using pip_ by running: ``pip install  django-simple-captcha``
2. Add ``captcha`` to the ``INSTALLED_APPS`` in your ``settings.py``
3. Run ``python manage.py syncdb`` (or ``python manage.py migrate`` if you are managing databae migrations via South) to create the required database tables
4. Add an entry to your ``urls.py``::

        urlpatterns += patterns('',
            url(r'^captcha/', include('captcha.urls')),
        )


.. _pip: http://pypi.python.org/pypi/pip

Adding to a Form
+++++++++++++++++

Using a ``CaptchaField`` is quite straight-forward:

Define the Form
----------------


To embed a CAPTCHA in your forms, simply add a ``CaptchaField`` to the form definition::

    from django import forms
    from captcha.fields import CaptchaField

    class CaptchaTestForm(forms.Form):
        myfield = AnyOtherField()
        captcha = CaptchaField()

…or, as a ``ModelForm``::


    from django import forms
    from captcha.fields import CaptchaField

    class CaptchaTestModelForm(forms.ModelForm):
        captcha = CaptchaField()
        class Meta:
            model = MyModel

Validate the Form
-----------------

In your view, validate the form as usually: if the user didn't provide a valid response to the CAPTCHA challenge, the form will raise a ``ValidationError``::

    def some_view(request):
        if request.POST:
            form = CaptchaTestForm(request.POST)

            # Validate the form: the captcha field will automatically
            # check the input
            if form.is_valid():
                human = True
        else:
            form = CaptchaTestForm()

        return render_to_response('template.html',locals())

Templates usage
---------------

Example with a refresh button :

.. image:: https://github.com/mothsART/django-simple-captcha/raw/master/docs/refresh.png

In your template.html file ::

    <html>
        <head>
        <style type="text/css" media="screen">
            #error_captcha{
                list-style-type: none;
            }
            #captcha_refresh{
                background-color: #ccc;
            }
            #captcha_refresh:hover{
                background-color: #888;
            }
            .captcha{
                border: solid 1px #888;
                position: absolute;
                margin-left: 10px;
                width: 150px;
                height: 30px;
                z-index: 1;
            }
            /* use a loading picture
            Because in a production site, load time isn't immediate...
            positioning (in css) below captcha picture.
            */
            .captcha_loading{
                position: absolute;
                margin-top: 8px;
                margin-left: 80px;
                z-index: 0;
            }
            #id_captcha_1{
                width: 200px;
                margin-left: 174px;
            }
        </style>
        </head>
        <body>
            <form method="post" action='.'>{% csrf_token %}
            {% for field in form %}
                {% if field.name = "captcha" %}
                    <ul id="error_captcha">
                    {% for error in field.errors %}
                        <li>{{ error|escape }}</li>
                    {% endfor %}
                    </ul>
                    <input type="button" id="captcha_refresh" value="refresh" />
                    <img class="captcha_loading" alt="captcha loading picture"
                    src="{{ STATIC_URL }}icons/loading.gif" />
                {% endif %}
                {{ field.label_tag }}
                {{ field }}
            {% endfor %}

            <input
            name="submit"
            type="submit"
            value="submit" />
            </form>
        <script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js"></script>
        <script type="text/javascript">
        $(document).ready(function (){
            // Use csrf with Ajax and Jquery
            // https://docs.djangoproject.com/en/dev/ref/contrib/csrf/#ajax
            jQuery(document).ajaxSend(function(event, xhr, settings) {
                function getCookie(name) {
                    var cookieValue = null;
                    if (document.cookie && document.cookie != '') {
                        var cookies = document.cookie.split(';');
                        for (var i = 0; i < cookies.length; i++) {
                            var cookie = jQuery.trim(cookies[i]);
                            // Does this cookie string begin with the name we want?
                            if (cookie.substring(0, name.length + 1) == (name + '=')) {
                                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                                break;
                            }
                        }
                    }
                    return cookieValue;
                }
                function sameOrigin(url) {
                    // url could be relative or scheme relative or absolute
                    var host = document.location.host; // host + port
                    var protocol = document.location.protocol;
                    var sr_origin = '//' + host;
                    var origin = protocol + sr_origin;
                    // Allow absolute or scheme relative URLs to same origin
                    return (url == origin || url.slice(0, origin.length + 1) == origin + '/') ||
                        (url == sr_origin || url.slice(0, sr_origin.length + 1) == sr_origin + '/') ||
                        // or any other URL that isn't scheme relative or absolute i.e relative.
                        !(/^(\/\/|http:|https:).*/.test(url));
                }
                function safeMethod(method) {
                    return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
                }

                if (!safeMethod(settings.type) && sameOrigin(settings.url)) {
                    xhr.setRequestHeader("X-CSRFToken", getCookie('csrftoken'));
                }
            });

            // Refresh captcha
            $('#captcha_refresh').click(function(){
                $.post("/captcha_reload/", function(data){
                    $('.captcha').filter(":first").attr('src', data.image_url);
                    $('#id_captcha_0').val(data.key);
                    $('#id_captcha_1').val('');
                });
            });
        });
        </script>
        </body>
    </html>
