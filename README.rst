=====
django-hashphrase
=====

django-hashphrase is a django module that facilitates the process that
users click on a link in an email and django handles the click.
Hashlink makes it easy to generate such a link, authenticate it or not,
calls a custom function or not, etc.

Quick start
-----------

1. Add "hashphrase" to your INSTALLED_APPS setting like this::

    INSTALLED_APPS = (
        ...
        'hashphrase',
    )

2. Include the hashphrase URLconf in your project urls.py like this::

    url(r'^hl/', include('hashphrase.urls')),

3. Also put this at the end of your urls.py like this::

    from hashphrase import hashphraseviews_autodiscover
    hashphraseviews_autodiscover() #autodiscover hashphraseviews.py in apps

3. To generate a link::

    from hashphrase import generate_hashphrase

    from django.contrib.auth.models import User
    any_object = User.objects.get(id=1)

    import datetime
    action = 'my_click_handler'
    hash_phrase = generate_hashphrase(request.user, any_object, datetime.datetime.now(), action=action)

    # Then generate for example "http://yourhost.com/hl/"+hash_phrase+"/"
    # that lick will call the "registered" function

4. To register a function, create a file called hashphraseviews.py and put the function in it.::

    from hashphrase import hashphrase_register
    @hashphrase_register('my_click_handler')  #name must match action when creating the link
    def test_success(request, has_error, error_code, hash_link, content_obj):
        """
        use hashphrase_register decorator to register this function to be called when
        users click on the email link.
        be sure to check has_error. If not verified, has_error = True
        See HashLink class for error code definition
        """
        if has_error or not hash_link or not content_obj:
            from hashphrase import HashLink
            ret = "Invalid email link."
            if error_code == HashLink.ERR_EXPIRED:
                ret = "Link expired."
            elif error_code == HashLink.ERR_INVALID_USER:
                ret = "Needs to login."
            elif error_code == HashLink.ERR_INVALID_LINK:
                ret = "Invalid link."
            return HttpResponse(ret)
        return HttpResponse("Successful.")

