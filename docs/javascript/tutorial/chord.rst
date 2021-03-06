Chord Socket
~~~~~~~~~~~~

This is an extension of the :doc:`MeshSocket <./mesh>` which syncronizes a common :js:class:`Object`. It works by providing some extra handlers to store data. This exposes the entire :py:class:`dict` API.

.. note::

    This is a fairly inefficient architecture for read intensive applications. For large databases which are infrequently changed, this is ideal. For smaller networks where there is significant access required, you should use the :doc:`sync <./sync>` socket.

Basic Usage
-----------

There are three limitations compared to a normal :js:class:`Object`.

1. Keys must be translatable to a :js:class:`Buffer`
2. Keys are automatically translated to a :js:class:`Buffer`
3. Fetching values is significantly slower than for a :js:class:`Object`

The only API differences between this and :js:class:`~js2p.mesh.MeshSocket` are for access to this dictionary. They are as follows.

:js:func:`~js2p.chord.ChordSocket.get`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A value can be retrieved by using the :js:func:`~js2p.chord.ChordSocket.get` method. These calls are about ``O(log(n))`` hops, as they approximately halve their search area with each hop.

.. code-block:: javascript

    > let foo = sock.get('test key', null);
    > console.log(foo)
    Promise { <pending> }
    > foo.then(console.log)  // prints the value, if it exists, or ``null``
    null

It is important to note that keys are all translated to :js:class:`Buffer` before being used, so it is required that you use a :js:class:`string` or :js:class:`Buffer`-like object.

:js:func:`~js2p.chord.ChordSocket.set`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A value can be stored by using the :js:func:`~js2p.chord.ChordSocket.set` method. Like the above, these calls are about ``O(log(n))`` hops, as they approximately halve their search area with each hop.

.. code-block:: javascript

    > sock.set('test key', 'value');  // Both of these calls are okay
    > sock.set(new Buffer('test key'), new Buffer('value'));

Like above, keys and values are all translated to :js:class:`Buffer` before being used, so it is required that you use a :js:class:`string` or :js:class:`Buffer`-like object.

:js:func:`~js2p.chord.ChordSocket.del`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This deletes an association. Like the above, this call is about ``O(log(n))``.

.. code-block:: javascript

    > sock.del('test')

:js:func:`~js2p.chord.ChordSocket.update`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The update method is simply a wrapper which updates based on a fed :js:class:`Object`. Essentially it runs the following:

.. code-block:: javascript

    > for (var key of update_dict)  {
    ... sock.set(key, update_dict[key]);
    ... }

:js:func:`~js2p.chord.ChordSocket.keys` / :js:func:`~js2p.chord.ChordSocket.values` / :js:func:`~js2p.chord.ChordSocket.items`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These methods are analagous to the ones in Python's :py:class:`dict`. There are three main differences:

1. They emulate the Python 3 behavior. So, they will still return an generator, rather than a list.
2. :js:func:`~js2p.chord.ChordSocket.values` will return a generator of :js:class:`Promise` s
3. :js:func:`~js2p.chord.ChordSocket.items` will return a generator of :js:class:`Buffer` :js:class:`Promise` pairs

:js:func:`~js2p.chord.ChordSocket.pop` / :js:func:`~js2p.chord.ChordSocket.popitem`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These methods are also analagous to the ones in Python's :py:class:`dict`. The main difference is that if the leasing system is active, calling this method may throw an error if you don't "own" whatever key is popped.

Events
------

In addition to the above, and those of :js:class:`~js2p.mesh.MeshSocket`, the :js:class:`~js2p.chord.ChordSocket` object has two :js:class:`Event` s.

First there's :js:func:`~js2p.chord.ChordSocket Event 'add'`. This is called whenever an association is added. Because the value is not necessarily stored by you, it is not given as an argument to this event.

.. code-block:: javascript

    > sock.on('add', (conn, key)=>{
    ... // conn is a reference to the socket
    ... console.log(`${key} was added`);
    ... });

This class has one other event: :js:func:`~js2p.chord.ChordSocket Event 'delete'`. This is called every time an association is removed.

.. code-block:: javascript

    > sock.on('delete', (conn, key)=>{
    ... console.log(`The association with key ${key} was deleted`);
    ... });

Advanced Usage
--------------

Refer to :doc:`the mesh socket tutorial <./mesh>`

Use In A Browser
----------------

Refer to :doc:`the mesh socket tutorial <./mesh>`
