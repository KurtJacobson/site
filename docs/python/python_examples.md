## Using the mouse to move widgets in GTK+

Here is a simple example demonstrating how one can move GTK+ widgets with the mouse.

We use the 'motion-notify' event to update the widget position. Before moving get 
the widgets size and position as well as its parents container size and check to make 
sure it does not get moved outside of its containers bounds.

### Result

<video autoplay loop>
  <source src="../images/move_btn.mp4" type="video/mp4" />
  Your browser does not support the video tag.
</video>

### Example Code

```python
#!/usr/bin/env python

import gi

gi.require_version('Gtk', '3.0')
from gi.repository import Gtk, Gdk
from gi.repository import GObject

class MoveButton(Gtk.Fixed):

    def __init__(self):
        Gtk.Fixed.__init__(self)

        # Initial event position
        self.initial_x = 0
        self.initial_y = 0

        # Initial button position
        self.initial_pos_x = 0
        self.initial_pos_y = 0


    def button_press_event(self, widget, event):

        # Get the parent window size
        pw = self.get_allocation().width
        ph = self.get_allocation().height

        # Get the button location and size
        x = self.child_get_property(widget, 'x')
        y = self.child_get_property(widget, 'y')
        w = widget.get_allocation().width
        h = widget.get_allocation().height

        # Calculate the maximum change in position such that the
        # button does not move outside the bounds of its parent
        self.dx_max = pw - (x + w)
        self.dy_max = ph - (y + h)

        # Save the initial values
        self.initial_x = event.x_root
        self.initial_y = event.y_root
        self.initial_pos_x = x
        self.initial_pos_y = y


    def motion_notify_event(self, widget, event):

        # Calculate the change in position of the pointer
        dx = event.x_root - self.initial_x
        dy = event.y_root - self.initial_y

        # Determine the new position such that the button
        # does not move outside the bounds of its parent
        x = self.initial_pos_x + max(min(dx, self.dx_max), -self.initial_pos_x)
        y = self.initial_pos_y + max(min(dy, self.dy_max), -self.initial_pos_y)

        # Actualy move the button
        self.child_set_property(widget, 'x', x)
        self.child_set_property(widget, 'y', y)


def make_button(text):

    # Helper function to make the buttons
    b = Gtk.Button.new_with_label(text)
    b.connect("button_press_event", move_btn.button_press_event)
    b.connect("motion_notify_event", move_btn.motion_notify_event)
    b.show()
    return b

# New top level window
window = Gtk.Window()
window.connect("destroy", Gtk.main_quit)

# Initialize the MoveButton container
move_btn = MoveButton()
window.add(move_btn)

# Add some buttons
move_btn.put(make_button("A button"), 50, 50)
move_btn.put(make_button("Another button"), 250, 100)

window.show_all()

# Start the main loop
Gtk.main()
```
