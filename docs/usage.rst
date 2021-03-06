Creating a *system tray icon*
-----------------------------

In order to create a *system tray icon*, the class ``pystray.Icon`` is used::

    import pystray

    icon = pystray.Icon('test name')


In order for the icon to be displayed, you must provide an icon. This icon must
be specified as a ``PIL.Image.Image``::

    from PIL import Image, ImageDraw

    def create_image():
        # Generate an image and draw a pattern
        image = Image.new('RGB', (width, height), color1)
        dc = ImageDraw.Draw(image)
        dc.rectangle(
            (width // 2, 0, width, height // 2),
            fill=color2)
        dc.rectangle(
            (0, height // 2, width // 2, height),
            fill=color2)

        return image

    icon.icon = create_image()


To finally show you icon, run the following code::

    icon.run()


The call to ``pystray.Icon.run()`` is blocking, and it must be performed from
the main thread of the application. The reason for this is that the *system tray
icon* implementation for *OSX* will fail unless called from the main thread, and
it also requires the application runloop to be running. ``pystray.Icon.run()``
will start the runloop.

The ``run()`` method accepts an optional argument: ``setup``, a callable.

The ``setup`` funciton will be run in a separate thread once the *system tray
icon* is ready. The icon does not wait for it to complete, so you may put any
code that would follow the call to ``pystray.Icon.run()`` in it.

The call to ``pystray.Icon.run()`` will not complete until ``stop()`` is called.


Getting input from the *system tray icon*
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In order to receive notifications about user interaction with the icon, a
popup menu can be added with the ``menu`` constructor argument.

This must be an instance of ``pystray.Menu``. Please see the reference for more
information about the format.

It will be displayed when the right-hand button has been pressed on the icon on
*Windows*, and when the icon has been clicked on other platforms. Menus are not
supported on *X*.

Menus also support a default item. On *Windows*, and *X*, this item will be
activated when the user clicks on the icon using the primary button. On other
platforms it will be activated if the menu contains no visible entries; it does
not have to be visible.

All properties of menu items, except for the callback, can be dynamically
calculated by supplying callables instead of values to the menu item
constructor. The properties are recalculated every time the icon is clicked or
any menu item is activated.

If the dynamic properties change because of an external event, you must ensure
that ``Icon.update_menu`` is called. This is required since not all supported
platforms allow for the menu to be generated when displayed.


Creating the menu
~~~~~~~~~~~~~~~~~

A menu consists of a list of menu items, optionally separated by menu
separators.

Separators are intended to group menu items into logical groups. They will not
be displayed as the first and last visible item, and adjacent separators will be
hidden.

A menu item has several attributes:

*text* and *action*
    The menu item text and its associated action.

    These are the only required attribute.

*checked*
    Whether the menu item is checked.

    This can be one of three values:

    ``False``
        The item is decorated with an unchecked check box.

    ``True``
        The item is decorated with a checked check box.

    ``None``
        There is no hint that the item is checkable.

    If you want this to actually be togglable, you must pass a callable that
    returns the current state::

        state = False

        def on_clicked(icon, item):
            global state
            state = not item.checked

        # Update the state in `on_clicked` and return the new state in
        # a `checked` callable
        Icon('test', create_image(), menu=Menu(
            MenuItem(
                'Checkable',
                on_clicked,
                checked=lambda item: state))).run()

*radio*
    Whether this is a radio button.

    This is used only if ``checked`` is ``True`` or ``False``, and only has a
    visual meaning. The menu has no concept of radio button groups::

        state = 0

        def set_state(v):
            def inner(icon, item):
                global state
                state = v
            return inner

        def get_state(v):
            def inner(item):
                return state == v
            return inner

        # Let the menu items be a callable returning a sequence of menu
        # items to allow the menu to grow
        Icon('test', create_image(), menu=Menu(lambda: (
            MenuItem(
                'State %d' % i,
                set_state(i),
                checked=get_state(i),
                radio=True)
            for i in range(max(5, state + 2))))).run()

*default*
    Whether this is the default item.

    It is drawn in a distinguished style and will be activated as the default
    item on platforms that support default actions. On *X*, this is the only
    action available.

*visible*
    Whether the menu item is visible.

*submenu*
    The submenu, if any, that is attached to this menu item.

    If this is set, the action will not be called.

Once created, menus and menu items cannot be modified. All attributes except for
the menu item callbacks can however be set to callables returning the current
value. This also applies to the sequence of menu items belonging to a menu: this
can be a callable returning the current sequence.
