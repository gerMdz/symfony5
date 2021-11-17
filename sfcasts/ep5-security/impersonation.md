# Impersonation: switch_user

Have you ever had a situation where you're helping someone online... and it would
be *so* much easier if you could see what *they're* seeing on their screen... or,
better, if you could temporarily take over and fix the problem yourself?

> Yeah. Just click the little paper clip icon to attach the file. It should
> be like near the bottom... a paper clip. What's "attaching a file"? Oh... it's
> um... like sending a "package"... but on the Internet.

Ah, memories. Symfony can't help teach your family how to attach files to an email.
But! It *can* help your customer service people via a feature called impersonation.
Very simply: this gives *some* users the superpower to temporarily log in as someone
else.

## Enabling the switch_user Authenticator

Here's how. First, we need to enable the feature. In `security.yaml`, under
our firewall somewhere, add `switch_user: true`.

This activates a new authenticator. So we now have our `CustomAuthenticator`,
`form_login`, `remember_me` and also `switch_user`.

How does it work? Well, we can now "log in" as anyone by adding `?_switch_user=`
to the URL and then an email address. Head back to the fixtures file -
`src/Fixtures/AppFixtures.php` - and scroll down. We have one other user whose
email we know - it's `abraca_user@example.com`. Copy that, paste it on the end of
the URL and... access denied!

Of course! We can't allow just *anyone* to do this. The authenticator will only allow
this if we have a role called `ROLE_ALLOWED_TO_SWITCH`. Let's give this to our admin
users. We can do this via `role_hierarchy`. Up here, `ROLE_ADMIN` has
`ROLE_COMMENT_ADMIN` and `ROLE_USER_ADMIN`. Let's also give them
`ROLE_ALLOWED_TO_SWITCH`.

And now... whoa! We switched users! That's a different user icon! And most
importantly, down on the web debug toolbar, we see `abraca_user@example.com`...
and it even shows who the original user is.

Behind the scenes, when we entered the email address in the URL, the `switch_user`
authenticator grabbed that and then leveraged our *user provider* to load that
`User` object. Remember: we have a user provider that knows how to load users from
the database by querying on their `email` property. So that's why we used `email`
in the URL.

To "exit" and go back to our original user, add `?_switch_user=` again with the
special `_exit`.

## Styling Changes During Impersonation

But before we do that, once a customer service person has switched to another account,
we want to make sure they don't *forget* that they switched. So let's add
a *very* obvious indicator to our page that we're currently "switched": let's make
this header background red.

Open the base layout: `templates/base.html.twig`. Up on top... find the `body` and
`nav`...  and I'll break this onto multiple lines. How can we check to see if we
are currently impersonating someone? Say `is_granted()` and pass this
`ROLE_PREVIOUS_ADMIN`. If you're impersonating someone, you *will* have this role.

In that case, add `style="background-color: red"`... with `!important`  to override
the nav styling.

Let's see it! Refresh and... ha! That's a *very* obvious hint that we're impersonating.

## Helping the User End Impersonation

To help the user *stop* impersonation, let's add a link. Go down to the dropdown
menu. Once again, check if `is_granted('ROLE_PREVIOUS_ADMIN')`. Copy the link below...
paste... then send the user to - `app_homepage` but pass an extra
`_switch_user` parameter set to `_exit`.

If you pass something to the second argument of `path()` that is *not* a wildcard
on the route, Symfony will set it as a query parameter. So this should give us
*exactly* what we want. For the text, say exit impersonation.

Try that! Refresh. It's obvious that we're impersonating... hit "exit impersonation"
and... we are back as `abraca_admin@example.com`. Sweet!

By the way, if you need more control over which users someone is allowed to switch
to, you can listen to the `SwitchUserEvent`. To prevent switching, throw an
`AuthenticationException`. We'll talk more about event listeners later.

Next: let's take a short break to do something *totally* fun, but... kind of not
related to security: build a user API endpoint.