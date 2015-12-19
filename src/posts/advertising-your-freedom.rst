((title . "Advertising Your Freedom")
 (date . "2015-12-14 23:40:02 +0100"))

One of the first things people starting out with Emacs do is disabling
the splash screen.  This is simple enough to do, all it requires is a
``(setq inhibit-splash-screen t)``.  Now, that's not the only place
Emacs uses to remind you of your freedoms.  If you pay attention to
the small things involved in startup, you'll surely have noticed that
it displays a more subtle notice in the echo area:

    For information about GNU Emacs and the GNU system, type C-h C-a.

Disabling that one is trickier.  The message itself originates from
``startup-echo-area-message`` which is invoked by
``display-startup-echo-area-message``.  A look at `the sources`_
reveals just how much more involved it is to rid Emacs of this
advertisement:

.. code:: elisp

    (defun display-startup-echo-area-message ()
      (let ((resize-mini-windows t))
        (or noninteractive                  ;(input-pending-p) init-file-had-error
            ;; t if the init file says to inhibit the echo area startup message.
            (and inhibit-startup-echo-area-message
                 user-init-file
                 (or (and (get 'inhibit-startup-echo-area-message 'saved-value)
                          (equal inhibit-startup-echo-area-message
                                 (if (equal init-file-user "")
                                     (user-login-name)
                                   init-file-user)))
                     ;; Wasn't set with custom; see if .emacs has a setq.
                     (condition-case nil
                         (with-temp-buffer
                           (insert-file-contents user-init-file)
                           (re-search-forward
                            (concat
                             "([ \t\n]*setq[ \t\n]+"
                             "inhibit-startup-echo-area-message[ \t\n]+"
                             (regexp-quote
                              (prin1-to-string
                               (if (equal init-file-user "")
                                   (user-login-name)
                                 init-file-user)))
                             "[ \t\n]*)")
                            nil t))
                       (error nil))))
            (message "%s" (startup-echo-area-message)))))

Looks like you'll either need to write something appearing as if it
were an invocation of the ``(setq inhibit-startup-echo-area-message
"<user>")`` form in your init file (and don't you dare putting it into
another file or a byte-compiled init file or a different user!) or
just redefine ``startup-echo-area-message`` to display something else.
Guess what option I went with...

I concur with `the variable's comment`_:

.. code:: elisp

    ;; FIXME? Why does this get such weirdly extreme treatment, when the
    ;; more important inhibit-startup-screen does not.
    (defcustom inhibit-startup-echo-area-message nil
      "Non-nil inhibits the initial startup echo area message.
    Setting this variable takes effect
    only if you do it with the customization buffer
    or if your init file contains a line of this form:
     (setq inhibit-startup-echo-area-message \"YOUR-USER-NAME\")
    If your init file is byte-compiled, use the following form
    instead:
     (eval \\='(setq inhibit-startup-echo-area-message \"YOUR-USER-NAME\"))
    Thus, someone else using a copy of your init file will see the
    startup message unless he personally acts to inhibit it."
      :type '(choice (const :tag "Don't inhibit")
                     (string :tag "Enter your user name, to inhibit"))
      :group 'initialization)

.. _the sources: http://git.savannah.gnu.org/cgit/emacs.git/tree/lisp/startup.el?id=23b5c22703eeee7b4fe6608ce12ffe3b87794933#n2153
.. _the variable's comment: http://git.savannah.gnu.org/cgit/emacs.git/tree/lisp/startup.el?id=23b5c22703eeee7b4fe6608ce12ffe3b87794933#n79
