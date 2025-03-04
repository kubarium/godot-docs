.. _doc_volumetric_fog:

Volumetric fog
==============

.. note::

    Volumetric fog is only supported in the Clustered Forward rendering backend,
    not Forward Mobile or Compatibility.

As described in :ref:`doc_environment_and_post_processing`, Godot supports
various visual effects including two types of fog: traditional (non-volumetric)
fog and volumetric fog. Traditional fog affects the entire scene at once and
cannot be customized with :ref:`doc_fog_shader`.

Volumetric fog can be used at the same time as non-volumetric fog if desired.

On this page, you'll learn:

- How to set up volumetric fog in Godot.
- What fog volumes are and how they differ from "global" volumetric fog.

.. seealso::

    The Godot demo projects repository contains a
    `volumetric fog demo <https://github.com/godotengine/godot-demo-projects/tree/4.0-dev/3d/volumetric_fog>`__.

Volumetric fog properties
-------------------------

After enabling volumetric fog in the WorldEnvironment node's Environment
resource, you can edit the following properties:

- **Density:** The base *exponential* density of the volumetric fog. Set this to
  the lowest density you want to have globally. FogVolumes can be used to add to
  or subtract from this density in specific areas. A value of ``0.0`` disables
  global volumetric fog while allowing FogVolumes to display volumetric fog in
  specific areas. Fog rendering is exponential as in real life. This means fog
  will never *fully* obscure objects regardless of distance.
- **Albedo:** The Color of the volumetric fog when interacting with lights. This
  multiplies lights' color within the volumetric fog. Mist and fog have an
  albedo close to white (``Color(1, 1, 1, 1)``) while smoke has a darker albedo.
  This does not affect fog color within FogVolumes.
- **Emission:** The emitted light from the volumetric fog. Even with emission,
  volumetric fog will not cast light onto other surfaces. Emission is useful to
  establish an ambient color. As the volumetric fog effect uses
  single-scattering only, fog tends to need a little bit of emission to soften
  the harsh shadows. This does not affect fog color within FogVolumes.
- **Emission Energy:** The brightness of the emitted light from the volumetric
  fog. This does not affect fog color within FogVolumes.
- **GI Inject:** Scales the strength of Global Illumination used in the
  volumetric fog's albedo color. A value of ``0.0`` means that Global
  Illumination will not impact the volumetric fog. This has a small performance
  cost when set above ``0.0``. FogVolumes do not display injected GI regardless
  of their albedo color.
- **Anisotropy:** The direction of scattered light as it goes through the
  volumetric fog. A value close to ``1.0`` means almost all light is scattered
  forward. A value close to ``0.0`` means light is scattered equally in all
  directions. A value close to ``-1.0`` means light is scattered mostly
  backward. Fog and mist scatter light slightly forward, while smoke scatters
  light equally in all directions.
- **Length:** The distance over which the volumetric fog is computed. Increase
  to compute fog over a greater range, decrease to add more detail when a long
  range is not needed. For best quality fog, keep this as low as possible.
- **Detail Spread:** The distribution of size down the length of the froxel
  buffer. A higher value compresses the froxels closer to the camera and places
  more detail closer to the camera.
- **Ambient Inject:** Scales the strength of ambient light used in the
  volumetric fog. A value of ``0.0`` means that ambient light will not impact
  the volumetric fog. This has a small performance cost when set above ``0.0``.
- **Sky Affect:** Controls how much volumetric fog should be drawn onto the
  background sky. If set to ``0.0``, volumetric fog won't affect sky rendering
  at all (including FogVolumes).

Two additional properties are offered in the **Temporal Reprojection** section:

- **Temporal Reprojection > Enabled:** Enables temporal reprojection in the
  volumetric fog. Temporal reprojection blends the current frame's volumetric
  fog with the last frame's volumetric fog to smooth out jagged edges. The
  performance cost is minimal, however it does lead to moving FogVolumes and
  Light3Ds "ghosting" and leaving a trail behind them. When temporal
  reprojection is enabled, try to avoid moving FogVolumes or Light3Ds too fast.
  Short-lived dynamic lighting effects should have **Volumetric Fog Energy** set
  to ``0.0`` to avoid ghosting.
- **Temporal Reprojection > Amount:** The amount by which to blend the last
  frame with the current frame. A higher number results in smoother volumetric
  fog, but makes "ghosting" much worse. A lower value reduces ghosting but can
  result in the per-frame temporal jitter becoming visible.

Light interaction with volumetric fog
-------------------------------------

To simulate fog light scattering behavior in real life, all light types will
interact with volumetric fog. How much each light will affect volumetric fog can
be adjusted using the **Volumetric Fog Energy** property on each light. Enabling
shadows on a light will also make those shadows visible on volumetric fog.

If fog light interaction is not desired for artistic reasons, this can be
globally disabled by setting **Volumetric Fog > Albedo** to a pure black color
in the Environment resource.

Balancing performance and quality
---------------------------------

There are a few project settings available to adjust volumetric fog performance
and quality:

- **Rendering > Environment > Volumetric Fog > Volume Size:** Base size used to
  determine size of froxel buffer in the camera X-axis and Y-axis. The final
  size is scaled by the aspect ratio of the screen, so actual values may differ
  from what is set. Set a larger size for more detailed fog, set a smaller size
  for better performance.
- **Rendering > Environment > Volumetric Fog > Volume Depth:** Number of slices
  to use along the depth of the froxel buffer for volumetric fog. A lower number
  will be more efficient but may result in artifacts appearing during camera
  movement.
- **Rendering > Environment > Volumetric Fog > Use Filter:** Enables filtering
  of the volumetric fog effect prior to integration. This substantially blurs
  the fog which reduces fine details but also smooths out harsh edges and
  aliasing artifacts. Disable when more detail is required.

Using fog volumes for local volumetric fog
------------------------------------------

Sometimes, you want fog to be constrained to specific areas. Conversely, you may
want to have global volumetric fog but fog should be excluded from certain
areas. Both approaches can be followed using FogVolume nodes.

Here's a quick start guide to using FogVolumes:

- Make sure **Volumetric Fog** is enabled in the Environment properties. If
  global volumetric fog is undesired, set its **Density** to ``0.0``.
- Create a FogVolume node.
- Assign a new FogMaterial to the FogVolume node's **Material** property.
- In the FogMaterial, set **Density** to a positive value to increase density
  within the FogVolume, or a negative value to subtract the density from global
  volumetric fog.
- Configure the FogVolume's extents and shape as needed.

.. note::

    Thin fog volumes may appear to flicker when the camera moves or rotates.
    This can be alleviated by increasing the
    **Rendering > Environment > Volumetric Fog > Volume Depth** project setting
    (at a performance cost) or by decreasing **Length** in the Environment
    volumetric fog properties (at no performance cost, but at the cost of lower
    fog range). Alternatively, the FogVolume can be made thicker and use a lower
    density in the **Material**.

FogVolume properties
--------------------

- **Extents:** The size of the FogVolume when **Shape** is **Ellipsoid**,
  **Cone**, **Cylinder** or **Box**. If **Shape** is **Cone** or **Cylinder**,
  the cone/cylinder will be adjusted to fit within the extents. Non-uniform
  scaling of cone/cylinder shapes via the **Extents** property is not supported,
  but you can scale the FogVolume node instead.
- **Shape:** The shape of the FogVolume. This can be set to **Ellipsoid**,
  **Cone**, **Cylinder, **Box** or **World** (acts as global volumetric fog).
- **Material:** The material used by the FogVolume. Can be either a
  built-in FogMaterial or a custom ShaderMaterial (:ref:`doc_fog_shader`).

After choosing **New FogMaterial** in the **Material** property, you can adjust
the following properties in FogMaterial:

- **Density:** The density of the FogVolume. Denser objects are more opaque, but
  may suffer from under-sampling artifacts that look like stripes. Negative
  values can be used to subtract fog from other FogVolumes or global volumetric
  fog.
- **Albedo:** The single-scattering Color of the FogVolume. Internally, member
  albedo is converted into single-scattering, which is additively blended with
  other FogVolumes and global volumetric fog's **Albedo**.
- **Emission:** The Color of the light emitted by the FogVolume. Emitted light
  will not cast light or shadows on other objects, but can be useful for
  modulating the Color of the FogVolume independently from light sources.
- **Height Falloff:** The rate by which the height-based fog decreases in
  density as height increases in world space. A high falloff will result in a
  sharp transition, while a low falloff will result in a smoother transition.
  A value of ``0.0`` results in uniform-density fog. The height threshold is
  determined by the height of the associated FogVolume.
- **Edge Fade:** The hardness of the edges of the FogVolume. A higher value will
  result in softer edges, while a lower value will result in harder edges.
- **Density Texture:** The 3D texture that is used to scale the member density
  of the FogVolume. This can be used to vary fog density within the FogVolume
  with any kind of static pattern. For animated effects, consider using a custom
  :ref:`fog shader <doc_fog_shader>`.

Custom FogVolume shaders
------------------------

This page only covers the built-in settings offered by FogMaterial. If you need
to customize fog behavior within a FogVolume node (such as creating animated fog),
FogVolume nodes' appearance can be customized using :ref:`doc_fog_shader`.
