.. _RegistrationIntro:

Registration Intro
==================

Learning Objectives
-------------------

It is essential to understand the key registration methods for CAS,
and also have an understanding of some of the large body of work done
in quantifying the likely error of a registration.

Upon completion of this section, the student will be able to:

* Understand what a coordinate system is
* Sketch the main coordinate systems in a CAS system
* Implement coordinate conversions using 4x4 homogeneous transformations
* Recall the main registration methods
* Understand some of the challenges when registering data to physical space


Introduction
------------

Registration is the process of aligning two `Coordinate Systems <../notebooks/coordinate_systems.html>`_.


Medical Image Computing
^^^^^^^^^^^^^^^^^^^^^^^

This example may be more familiar to you if you have done the `IPMI`_ course.
In medical imaging terms, registration is often done to align image-volumes, e.g. align MR to CT

Like in this example, shown on the `AnalyzeDirect channel on YouTube <https://www.youtube.com/channel/UCbHc7Ec9_SQ8j7RAXF3rO3A>`_:

.. raw:: html

  <iframe width="560" height="315" src="https://www.youtube.com/embed/PDgBxvi1GdQ" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


Most medical image viewers provide similar functionality to align 3D volumes.


Computer Assisted Surgery
^^^^^^^^^^^^^^^^^^^^^^^^^

In CAS, the problem also exists in intra-device, inter-device or image-device scenarios:

* **image-device**: registering a pre-operative volume image to a tracker.
* **inter-device**: registering a camera coordinate system to a tracker
* **intra-device**: registering 2 poses of a camera at subsequent time points

For example:

* **image-device**: registering pre-operative data (CT/MR) scans to patient (tracker/world) space, to display the physical location of the tip of a tracked pointer in the MR/CT scan.
* **inter-device**: registering pre-operative data (CT/MR) scans to a laparoscopic video feed. This can be done directly, by matching the CT/MR coordinates to the video camera coordinates [Espinel2020]_, or indirectly by registering CT/MR to tracker space, and then using tracking and calibration information to work out where the camera is, and hence where the CT/MR is relative to the camera [Thompson2015]_.
* **intra-device**: registering feature points in one video frame to the next, and working out the difference in camera position which would enable triangulating those points.


Methods
-------

Typically, methods in CAS, are sub-divided (e.g. in :ref:`bookPeters`) into:

* Manual
* Point-based
* Surface-based (also called Shape-based)
* Volume-based, (i.e. intra-op CT to pre-op CT, not covered, see [Octay2013]_.)
* Calibration-based, covered earlier as examples [Feuerstein2008]_, [Kang2014]_.

These are covered in the next sections.


A Note on Coordinate Systems and Rotations
------------------------------------------

A brief introduction to coordinate transformations is provided in the accompanying :ref:`Notebooks`.

In 3D space, we typically consider 6 degrees-of-freedom (DOF):

* Translations along x, y, z cartesian axis = 3 DOF
* Rotations about x, y, z cartesian axis = 3 DOF

So, registration and converting coordinates from one
coordinate system to another require understanding of how these work.

However:

* There are `several rotation formulations`_.
* Euler angles get confusing when you consider `extrinsic or intrinsic`_ rotation.
* Euler angles, Quaternions, Rodrigues (axis-angle) representation (see above links), can be converted between each other, and to a 3x3 rotation matrix.
* Rotation matrices are not commutative
* The preferences around ordering of rotation matrices and especially when discussing Euler Angles, is software/application/community/culture specific.
* Note that the underlying graphics system may use a different convention to a higher level software API.
* Assume NOTHING. Every time you implement these things, start with a very clear definition of what you are meant to be implementing.


A Note on VTK Coordinate Systems
--------------------------------

* Several pieces of software, including `Slicer`_, `MITK`_, `PLUS`_, `NifTK`_, `scikit-surgery`_ all use VTK.
* Look in `vtkProp3D <https://gitlab.kitware.com/vtk/vtk/blob/master/Rendering/Core/vtkProp3D.cxx#L163>`_, and at ``SetOrientation()`` which says *"Orientation is specified as X,Y and Z rotations in that order, but they are performed as RotateZ, RotateX, and finally RotateY"*.
* vtkProp3D therefore suggests that VTK uses *"Tait–Bryan angles"*, specifically the z-x-y option, which are therefore **intrinsic** rotations meaning, they move with the object being moved.

This has been implemented in the `SciKit-Surgery`_ platform, specifically:

* This matrix construction has been implemented in `scikit-surgerycore <https://github.com/UCL/scikit-surgerycore/blob/master/sksurgerycore/transforms/matrix.py>`_
* The *standard* VTK ordering has been implemented in `scikit-surgeryvtk <https://github.com/UCL/scikit-surgeryvtk/blob/master/sksurgeryvtk/utils/matrix_utils.py#L50>`_.

In addition:

* In `vtkTransform <https://gitlab.kitware.com/vtk/vtk/blob/master/Common/Transforms/vtkTransform.h#L92>`_, there is a method ``RotateWXYZ()`` which sets the rotation as an angle about a world axis. Internally, this uses quaternions and converts the world axis to a homogeneous matrix. This is an **extrinsic** rotation.


A Note on Homogeneous Coordinate Conventions
--------------------------------------------

As is common (e.g. `euclideanspace.com`_, `brainvoyager`_, `opengl`_) we represent

* rotations as the upper-left 3x3 matrix in a 4x4 homogeneous transformation matrix.
* translation as the right-most 3x1 vector in a 4x4 homogeneous transformation matrix.

Note the comment on the tutorial on the `opengl`_ website: *"This is the single most important
tutorial in the whole set. Be sure to read it at least 8 times"*.

This is not being facetious. It is good advice.

.. _`several rotation formulations`: https://en.wikipedia.org/wiki/Rotation_formalisms_in_three_dimensions
.. _`extrinsic or intrinsic`: https://en.wikipedia.org/wiki/Euler_angles#Extrinsic_rotations
.. _`Tait–Bryan angles`: https://en.wikipedia.org/wiki/Euler_angles#Extrinsic_rotations
.. _`euclideanspace.com`: https://www.euclideanspace.com/maths/geometry/affine/matrix4x4/index.htm
.. _`brainvoyager`: https://www.brainvoyager.com/bv/doc/UsersGuide/CoordsAndTransforms/SpatialTransformationMatrices.html
.. _`opengl`: http://www.opengl-tutorial.org/beginners-tutorials/tutorial-3-matrices/
.. _`Slicer`: https://www.slicer.org/
.. _`MITK`: http://www.mitk.org
.. _`PLUS`: https://plustoolkit.github.io/
.. _`NifTK`: http://www.niftk.org
.. _`SciKit-Surgery`: https://github.com/UCL/scikit-surgery/wiki
.. _`IPMI`: https://ucl.reportlab.com/modules/MPHY0025/pdf/