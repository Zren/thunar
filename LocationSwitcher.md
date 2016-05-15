Issue: https://bugzilla.xfce.org/show_bug.cgi?id=12578

I took a dive into the code to see how hard this would be. So here's some notes.

If the container can handle the click event, then it 50% easier, because otherwise you have to figure out the positioning since the child widgets are manually positioned.


thunar-window.c
	Probably needs a function similar to the "view-location-selector-toolbar" action handler that changes to the texfield but doesn't update gtk's config. This is needed so that when we lose focus of the textfield, we can tell if we need to change back to buttons or not.

	"view-location-selector-pathbar"
		action to change to the location buttons
	"view-location-selector-toolbar"
		action to change to the location textfield

thunar-location-entry.c
	Needs to handle whatever the GTK loseFocus/blur event is to switch back to the buttons. This can simply call the "view-location-selector-pathbar" action handler.

thunar-location-buttons.c
	Needs a button to call the "view-location-selector-toolbar" action (or our new action/function that doesn't update the config).
	If we can "click" the background, this will be really easy, so long as the child buttons of the container block the click event from bubbling.
	If not, we need a new button between the last button and the "scroll" button called buttons->right_slider.
		Note that buttons can be RTL or LTR
		Copy the buttons->right_slider code.
		LTR
			x1 is 
			x2 is right_slider_offset
		RTL
			x1 is right_slider_offset + buttons->slider_width
			x2 is
		and (child_allocation.x = right_slider_offset + allocation->x;) is probably needed too to draw properly (allocation->x is the container's offset).



	thunar_location_buttons_size_allocate(...)
		this is where the button's are positioned and the slider buttons are toggled on/off
		this is where we would need to resize the button that calls "view-location-selector-toolbar"

