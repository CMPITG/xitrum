Writing postbacks
=================

.. image:: http://www.bdoubliees.com/journalspirou/sfigures6/schtroumpfs/s2.jpg

Please see the following links for the idea about postback:

* http://en.wikipedia.org/wiki/Postback
* http://nitrogenproject.com/doc/tutorial.html

Xitrum supports Ajax form postback, with additional features:

* Anti-CSRF
* :doc:`Validation </validation>`

Form
----

ArticleShow.scala

::

  @GET("/articles/:id")
  class ArticleShow {
    override def execute {
      val id = param("id")
      val article = Article.find(id)
      renderView(
        <h1>{article.title}</h1>
        {article.body}
      )
    }
  }

ArticleNew.scala

::

  // first=true: force this route to be matched before "/articles/:id"
  @GET(value="/articles/new", first=true)
  class ArticleNew extends Action {
    override def execute {
      renderView(
        <form postback="submit" action={urlForPostbackThis}>
          Title:
          <input type="text" name="title" /><br />

          Body:
          textarea name="body"></textarea><br />

          <input type="submit" value="Save" />
        </form>
      )
    }

    override def postback {
      val title   = param("title")
      val body    = param("body")
      val article = Article.save(title, body)

      flash("Article has been saved.")
      jsRedirectTo[ArticleShow]("id" -> article.id)
    }
  }

When ``submit`` JavaScript event of the form is triggered, the form will be posted back
to the current Xitrum action.

``action`` attribute of ``<form>`` is encrypted. The encrypted URL acts as the anti-CSRF token.

Non-form
--------

Postback can be set on any element, not only form.

An example with link:

::

  <a href="#" postback="click" action={urlForPostback[LogoutAction]}>Logout</a>

Clicking the link above will trigger the postback to LogoutAction.

Confirmation dialog
-------------------

If you want to display a confirmation dialog:

::

  <a href="#" postback="click"
              action={urlForPostback[LogoutAction]}
              confirm="Do you want to logout?">Logout</a>

If the user clicks "Cancel", the postback will not be sent.

Additional params
-----------------

In case of form, you can add ``<input type="hidden"...`` elements to send
additional params with the postback.

For other elements, you do like this:

::

  <a href="#" postback="click"
              action={urlForPostbackThis("itemId" -> item.id)}
              confirm={"Do you want to delete %s?".format(item.name)}>Delete</a>
