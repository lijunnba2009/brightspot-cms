Page Layouts
------------

Brightspot implements a subset of the W3 Grid Layout specification for defining grid layouts. This implementation provides a flexible way to define grid layouts using a compact CSS format.

Using the <cms:render> JSP Tag, content can be rendered into the CSS grid items defined by CSS layouts.

Overview
~~~~~~~~

Grid layouts are defined in regular CSS files. Brightspot will automatically read all CSS files in the project and find any defined grid layouts.

For each content type being rendered, there should be a grid layout defined in CSS, a JSP to use the CSS grid layout, and a JSP to render the content.

Brightspot does not require the use of CSS grid layouts, but it makes defining layouts much easier.

Creating a Layout
~~~~~~~~~~~~~~~~~

Grid layouts are defined in CSS files contained in your project directories.

Brightspot implements the following CSS properties defined in the W3 Grid Layout specification: display: -dari-grid, -dari-grid-template, -dari-grid-definition-columns, -dari-grid-definition-rows

Grid lengths allow the following units: percentages, em, px, auto and fr.

Grids defined in CSS should use class selectors (.grid) instead of id selectors (#grid).

An example grid:

::

    .layout-global {
        display: -dari-grid;
        -dari-grid-template: ".    header  .   "
                            "main  main   main"
                            ".    footer  .   ";

        -dari-grid-definition-columns: 1fr 1140px 1fr;
        -dari-grid-definition-rows: 158px auto auto;

        margin-bottom: 10px;
    }

A -dari-grid-template is similar to a table. For this version there are three columns, and three rows. The header is in the middle column, and the main area spans all three. The footer, like the header, is in the middle. The -dari-grid-definition-columns define the widths of each column and the -dari-grid-definition-rows define the height.

Placing Content
~~~~~~~~~~~~~~~

Grids can be used to lay out pages in JSPs using the <cms:layout> JSP tag. The tag takes the name of the CSS class selector that defines a grid and uses it to output the appropriate <div> tags to alter the layout.

Content can be placed inside of named or numbered grid items using the <cms:render> JSP tag.

.. code-block:: jsp

    <cms:layout class="layout-global">
        <cms:render area="header">
            Header Area
        </cms:render>
        <cms:render area="main">
            Main
        </cms:render>
        <cms:render area="footer">
            Footer
        </cms:render>
    </cms:layout>

It is also possible to render content objects directly using the <cms:render> tag.

To view your grid on a page you have created, add ?_prod=false&_grid=true to the end of the URL.

.. code-block:: jsp

    <cms:render value="${mainContent}" area="main">

Content can be rendered from the object or the template. For example, where the header and footer objects are defined on the template:

.. code-block:: jsp

    <cms:layout class="layout-global">
        <cms:render value="${header}" area="header"/>
        <cms:render value="${mainContent}" area="main"/>
        <cms:render value="${footer}" area="footer"/>
    </cms:layout>

Brightspot UI Layouts
~~~~~~~~~~~~~~~~~~~~~

In addition to providing an easy to implement front-end solution, the grid spec is also used in Brightspot, where it can be used to build out an editorial interface representative of the front-end display. This makes it easy for developers to define content that can be placed in areas of the grid and for editors to build the pages out.

In this example, a page is going to be created for an example site. The goal is to create a page that allows an editor to place modules in three layouts. The layouts can be combined to build out the entire page:

* Single Module: A single module spanning the entire layout width. 
* Two Modules: Two modules side-by-side, spanning the entire layout width.
* Three Modules: Three modules side-by-side, spanning the entire layout width.

A set of applicable modules that can be used on the page can be grouped together using an interface.

Creating the page:
^^^^^^^^^^^^^^^^^^

Start by creating the Java class that will be the model for your page. Extend Content and add two annotations: one to render the page, the page-container.jsp, and the other to render the content in the page object, the page-object.jsp

.. code-block:: java

    @Renderer.LayoutPath("/WEB-INF/common/page-container.jsp")
    @Renderer.Path("/WEB-INF/model/page-object.jsp")
    public class YourPage extends Content {

        private String name;

        // Getters and Setters

    }

Create the three layouts:
^^^^^^^^^^^^^^^^^^^^^^^^^

Use the grid layout spec to build out the three layouts. The first has a grid area 0 representing the single module spanning the page.

::

    .grid-1-module {
        display: -dari-grid;
        -dari-grid-template:
            "0";

        -dari-grid-definition-columns: 940px;
        -dari-grid-definition-rows: auto;
        padding-bottom: 30px;

    }

The second layout adds a new grid area, 1, to contain the second module. Adjust the px and margins to make room for the other module.

::

    .grid-2-modules {
        display: -dari-grid;
        -dari-grid-template:
            "0 . 1";

        -dari-grid-definition-columns: 460px 20px 460px;
        -dari-grid-definition-rows: auto;
        margin-bottom: 30px;
    }

The third layout adds another grid area, 2, and adjusts the spacing to make all three areas the same width across the page. The ListLayout processes the grid areas in order. Always add new areas in numerical order, if you use numbers.

::

    .grid-3-modules {
        display: -dari-grid;
        -dari-grid-template:
            "0 . 1 . 2";

        -dari-grid-definition-columns: 300px 20px 300px 20px 300px;
        -dari-grid-definition-rows: auto;
        margin-bottom: 30px;
    }

Add Layouts to Page:
^^^^^^^^^^^^^^^^^^^^

Once grid layouts have been defined, add them as options for an editor to choose. The actual CSS class names must be used as the grid layouts in the Java class.

Add a list of the content types that can be placed into the defined areas.

In the example below, Placeable.class is an interface that any addable module implements. For the defined grid areas for a layout, specify the content that can be added. In the example, ImageModule.class is the only module that can be placed in the one wide grid layout. Also, the third module area in the three wide layout must always be a TextModule.class. These modules implement Placeable.class, so they can also be added in the other areas.

.. code-block:: java

    @Renderer.LayoutPath("/WEB-INF/common/page-container.jsp")
    @Renderer.Path("/WEB-INF/model/page-object.jsp")
    public class YourPage extends Content {

        @Required
        private String name;

        @ToolUi.Heading("Modules Grid")
            @Renderer.ListLayouts(map={

            @Renderer.ListLayout(name="grid-1-module",
            itemClasses={ImageModule.class}),

            @Renderer.ListLayout(name="grid-2-modules",
            itemClasses={Placeable.class, Placeable.class}),

            @Renderer.ListLayout(name="grid-3-modules",
            itemClasses={Placeable.class, Placeable.class, TextModule.class})    

            })

        private List<Placeable> modules;

        // Getters and Setters

Render the Layout:
^^^^^^^^^^^^^^^^^^

To render the content, use the <cms:layout> tag to render the list of content in the defined area.

.. code-block:: jsp

    <cms:layout class="${cms:listLayouts(content, 'modules')}">
        <cms:render value="${content.modules}" />
    </cms:layout>

Mobile Devices
~~~~~~~~~~~~~~

To modify the layout dynamically based on screen size, you can use a @media query to override the layout class. In the example below, once the screen size drops below 700px, the right rail content appears below the main content and the content width is reduced.

::

    .layout {
        display: -dari-grid;
        -dari-grid-template: ". main . right .";

        -dari-grid-definition-columns: 1fr 680px 80px 360px 1fr;
        -dari-grid-definition-rows: auto;

        padding: 10px;
        }

    @media only screen and (min-width: 300px) and (max-width:700px) {
            .layout {
            display: -dari-grid;
            -dari-grid-template: ".  main    ."
                                ".  right   .";

            -dari-grid-definition-columns: 1fr 320px 1fr;
            -dari-grid-definition-rows: auto auto;

            padding: 10px;
        }
    }

Context
~~~~~~~

You may need to determine what grid area a piece of content is being placed into. For example, an image could be placed into various sizes of grid area, so a crop size for each area would need to be set. In order to allow context to be determined, each grid area can have context set:

::

    .grid-3-modules {
        display: -dari-grid;
        -dari-grid-template:
            "0 . 1 . 2";

        -dari-grid-definition-columns: 325px 18px 325px 18px 325px;
        -dari-grid-definition-rows: auto;
        -dari-grid-contexts: 0 grid-3-modules 1 grid-3-modules 2 grid-3-modules;
    }

    .grid-2-modules {
        display: -dari-grid;
        -dari-grid-template:
            "0 . 1";

        -dari-grid-definition-columns: 497px 16px 497px;
        -dari-grid-definition-rows: auto;
        -dari-grid-contexts: 0 grid-2-modules 1 grid-2-modules;
    }

This allows context to be checked in the JSP and the correct crop set:

.. code-block:: jsp

    <c:if test="${cms:inContext('grid-3-modules')}">
        <c:set var="size" value="threeWideCrop" />
    </c:if>
    <c:if test="${cms:inContext('grid-2-modules')}">
        <c:set var="size" value="twoWideCrop" />
    </c:if>
    <cms:img overlay="true" src="${content.image}" size="${size}"/>

Context can also be checked on the object render level:

.. code-block:: java

    @Renderer.Paths ({
    @Renderer.Path(value = "/WEB-INF/common/image.jsp"),
    @Renderer.Path(context = "grid-3-modules",
        value = "/WEB-INF/modules/image-three-wide.jsp")
    @Renderer.Path(context = "grid-2-modules",
        value = "/WEB-INF/modules/image-two-wide.jsp")
    })
    @Renderer.LayoutPath("/WEB-INF/common/page-container.jsp")
    public class ImageModule extends Content {


    }

Tags
~~~~

API Definitions for JSP tags used in the grid layout system.

<cms:layout>
^^^^^^^^^^^^

class - Name of the CSS class that defines a grid.

<cms:render>
^^^^^^^^^^^^

The render tag will render the contents of the value attribute into a provided area.

area - Name of the area to render content into.

value - Value to render. This can be Content, ReferentialText, an Iterable, or a String.

