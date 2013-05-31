Implementing an undo-redo-manager in Python
===========================================
:date: 2013-05-31
:category: python
:summary: This post explains one way of implementing a class that is able
          to undo and redo each called command. The explained technique
          will be used in my GSOC project, but I will describe it here in
          a general way so that it can be adapted to your needs.

An undo-redo-manager is useful in many cases, especially in interactive
applications (interactive shells, grahical applications like GIMP_ or
LibreOffice_). There are two main different implementation types to
consider: `save state and generate state`_.  Save state means that each
performed action results in “freezing” the current state with all its
relevant information and saving this state. If a certain state has to be
recovered, the current state is swapped out with the desired state and the
new state is loaded. Generate state, on the other hand, means that every
action has its own undo action. This approach uses two stacks: one for the
undo actions and one for the redo actions. After an action has been
executed, it is pushed onto the undo stack and the redo stack is cleared
to make sure it contains no elements after a new command has been
executed. To undo the last action, the last recently pushed element from
the undo stack is poppped and its undo method is executed. After that, the
command is pushed onto the redo stack. To redo the last undone action, the
last recently pushed element from the redo stack is popped and executed.
After that, it is pushed onto the undo stack.

Now that I have told you that there are two different ways to implement
such a manager, you may ask yourself: “What is the better approach?” As
(almost) always, the answer is: it depends. The generate state is used for
commands that can easily be undone using an "opposite" command. An example
for this are the text formatting operations of LibreOffice Writer: the
opposite action of “increase font size by 12pt” is “decrease font size by
12pt” and so on. But there are also applications where it is hard or
impossible to generate an opposite operation of a certain action. Think of
image manipulation programs like GIMP: How do you sharpen an image so that
it looks like the original after it has been blurred?

In this post I will demonstrate an implementation that uses **generate
state**.

In my example, I have two commands: one adds an element to a set and one
removes an element from a set. That means we will have two classes:
``ElementAdder`` for adding an element to a set and ``ElementRemover`` for
removing an element from a set. Both have a ``__call__`` and an ``undo``
method to execute the command or its undo command, respectively. I
realized that both these method are ones that operate on a given set and
use an element to operate on this set. So I wrote a base class
``SetOperation`` with the abstract methods ``__call__`` and ``undo``. Its
``__init__`` method receives a set and an element. This makes it easy to
add another command, e.g. ``SilentElementRemover`` which uses the
``discard`` method of the set instead of ``remove``.

.. code-block:: py

   from abc import ABCMeta, abstractmethod
   
   
   class SetOperation(object):
       __metaclass__ = ABCMeta
   
       def __init__(self, set_, element):
           self.set_ = set_
           self.element = element
   
       @abstractmethod
       def __call__(self):
           return
   
       @abstractmethod
       def undo(self):
           return
   
   
   class ElementAdder(SetOperation):
       def __call__(self):
           self.set_.add(self.element)
   
       def undo(self):
           self.set_.remove(self.element)
   
   
   class ElementRemover(SetOperation):
       def __call__(self):
           self.set_.remove(self.element)
   
       def undo(self):
           self.set_.add(self.element)

The ``CommandManager`` class manages all executed commands by saving them
in an undo and a redo stack. The ``push_*`` and ``pop_*`` methods are
trivial and are sufficiently explained in their corresponding docstring.
The ``do`` method is for calling a new command. Apart from of course
calling the passed command, this method also pushes this command onto the
undo stack and clears the redo stack, as explained above. The ``undo``
method receives an optional integer (default: 1) as a parameter whic
indicates the number of commands which have to be undone. If not all
commands can be undone, an exception is raised as soon as this happens
(imagine attempting to undo 5 actions if you have only done 2). On every
iteration, the top element from the undo stack is popped, its undo method
is called and pushed to the redo stack. The ``redo`` method receives an
optional integer as well, again defaulting to 1. On every iteration, the
top element from the redo stack is popped, executed and pushed onto the
undo stack. Note that this method does not clear the redo stack, like the
method ``do`` does.

.. code-block:: py

    class CommandManager(object):
        def __init__(self):
            self.undo_commands = []
            self.redo_commands = []
    
        def push_undo_command(self, command):
            """Push the given command to the undo command stack."""
            self.undo_commands.append(command)
    
        def pop_undo_command(self):
            """Remove the last command from the undo command stack and return it.
            If the command stack is empty, EmptyCommandStackError is raised.
    
            """
            try:
                last_undo_command = self.undo_commands.pop()
            except IndexError:
                raise EmptyCommandStackError()
            return last_undo_command
    
        def push_redo_command(self, command):
            """Push the given command to the redo command stack."""
            self.redo_commands.append(command)
    
        def pop_redo_command(self):
            """Remove the last command from the redo command stack and return it.
            If the command stack is empty, EmptyCommandStackError is raised.
    
            """
            try:
                last_redo_command = self.redo_commands.pop()
            except IndexError:
                raise EmptyCommandStackError()
            return last_redo_command
    
        def do(self, command):
            """Execute the given command. Exceptions raised from the command are
            not catched.
    
            """
            command()
            self.push_undo_command(command)
            # clear the redo stack when a new command was executed
            self.redo_commands[:] = []
    
        def undo(self, n=1):
            """Undo the last n commands. The default is to undo only the last
            command. If there is no command that can be undone because n is too big
            or because no command has been emitted yet, EmptyCommandStackError is
            raised.
    
            """
            for _ in xrange(n):
                command = self.pop_undo_command()
                command.undo()
                self.push_redo_command(command)
    
        def redo(self, n=1):
            """Redo the last n commands which have been undone using the undo
            method. The default is to redo only the last command which has been
            undone using the undo method. If there is no command that can be redone
            because n is too big or because no command has been undone yet,
            EmptyCommandStackError is raised.
    
            """
            for _ in xrange(n):
                command = self.pop_redo_command()
                command()
                self.push_undo_command(command)

Let's see the code in action! Here is a copy of an interactive python
session:

.. code-block:: pycon

    >>> my_set = {3, 5, 13}
    >>> my_set
    set([3, 13, 5])
    >>> manager = CommandManager()
    >>> # remove element 5 from the set
    >>> manager.do(ElementRemover(my_set, 5))
    >>> my_set
    set([3, 13])
    >>> # add element -7 to the set
    >>> manager.do(ElementAdder(my_set, -7))
    >>> my_set
    set([-7, 3, 13])
    >>> # undo adding element -7
    >>> manager.undo()
    >>> my_set
    set([3, 13])
    >>> # undo removing element 5
    >>> manager.undo()
    >>> my_set
    set([3, 13, 5])
    >>> # redo both undone operations
    >>> manager.redo(2)
    >>> my_set
    set([-7, 3, 13])

The complete code sample can be found at undoredomanager.py_ from my
hodgepodge GitHub repository.

.. _save state and generate state: http://stackoverflow.com/a/3542670/657575
.. _GIMP: http://www.gimp.org/
.. _LibreOffice: http://www.libreoffice.org/
.. _undoredomanager.py: https://github.com/derdon/hodgepodge/blob/master/python/undoredomanager.py
