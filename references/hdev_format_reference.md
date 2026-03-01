# HDevelop (`.hdev`) Format Reference

HDevelop uses XML-based `.hdev` files. The HALCON operator calls, control flow, and data model are identical to HDevelopEVO's `.hscript` — only the file format differs.

## File types

| Extension | Purpose | Structure |
|-----------|---------|-----------|
| `.hdev` | Script (program) | XML with one or more `<procedure>` elements; entry point is `<procedure name="main">` |
| `.hdvp` | Single external procedure | XML with exactly one `<procedure>` element (no `main`) |
| `.hdpl` | Procedure library | XML with `<library/>` marker and multiple `<procedure>` elements |

All three share the same XML schema, differing only in the presence of `<library/>` and the number of procedures.

## XML schema overview

```xml
<?xml version="1.0" encoding="UTF-8"?>
<hdevelop file_version="1.2" halcon_version="25.11.0.0">

  <!-- Optional: marks the file as a procedure library (.hdpl only) -->
  <library/>

  <procedure name="main">
    <interface/>          <!-- parameter declarations (empty for main) -->
    <body>
      <c>* comment line</c>
      <l>read_image (Image, 'fabrik')</l>
      <l>threshold (Image, Region, 128, 255)</l>
    </body>
    <docu id="main">      <!-- documentation block (required, even for main) -->
      <abstract lang="en_US">Description of what the script does.</abstract>
      <abstract lang="de_DE">Beschreibung, was das Skript tut.</abstract>
      <short lang="en_US">One-line summary.</short>
      <short lang="de_DE">Einzeilige Zusammenfassung.</short>
      <parameters/>
    </docu>
  </procedure>

  <!-- additional procedures in the same file -->
  <procedure name="helper_proc" access="local">
    ...
  </procedure>

</hdevelop>
```

## Root element: `<hdevelop>`

| Attribute | Value | Description |
|-----------|-------|-------------|
| `file_version` | `"1.2"` | XML format version |
| `halcon_version` | `"25.11.0.0"` | HALCON version the file was created with |

## `<library/>` element

Empty self-closing tag. Present only in `.hdpl` files to mark the file as a procedure library. Must appear before any `<procedure>` elements.

## `<procedure>` element

| Attribute | Required | Values | Description |
|-----------|----------|--------|-------------|
| `name` | yes | string | Procedure name (`"main"` for the entry point) |
| `access` | no | `"local"` | If set to `"local"`, the procedure is hidden from external callers (used in `.hdpl` files for internal helpers) |

Children (in order):
1. `<interface>` — parameter declarations
2. `<body>` — executable code
3. `<docu>` — documentation (**required** for every procedure, including `main`)

## `<interface>` element

Defines the procedure's input/output parameters. Can be empty (`<interface/>`) for parameterless procedures.

Contains up to four child groups, each holding `<par>` elements:

| Tag | Direction | Type | HScript equivalent |
|-----|-----------|------|--------------------|
| `<io>` | input | iconic (image/region/XLD) | `object` |
| `<oo>` | output | iconic | `out object` |
| `<ic>` | input | control (number/string/handle) | `tuple` |
| `<oc>` | output | control | `out tuple` |

### `<par>` element (parameter)

| Attribute | Values | Description |
|-----------|--------|-------------|
| `name` | string | Parameter name |
| `base_type` | `"iconic"` or `"ctrl"` | Iconic = image/region/XLD; ctrl = number/string/handle |
| `dimension` | `"0"`, `"1"`, `"2"`, ... | Vector nesting depth (0 = scalar/tuple, 1 = vector, 2 = vector of vectors) |

### Example: procedure with all four parameter groups

```xml
<procedure name="find_qr_codes">
<interface>
<io>
  <par name="Image" base_type="iconic" dimension="0"/>
</io>
<oo>
  <par name="QrCodeXlds" base_type="iconic" dimension="0"/>
</oo>
<ic>
  <par name="QrCodeModel" base_type="ctrl" dimension="0"/>
</ic>
<oc>
  <par name="QrCodeHandles" base_type="ctrl" dimension="0"/>
  <par name="QrCodeStrings" base_type="ctrl" dimension="0"/>
</oc>
</interface>
```

## `<body>` element

Contains the executable code as a sequence of two element types:

| Tag | Content | Description |
|-----|---------|-------------|
| `<l>` | HDevelop code | One line of executable code |
| `<c>` | `* comment text` | One comment line (always starts with `*`) |

### XML character escaping

Standard XML escaping applies inside `<l>` and `<c>` elements:

| Character | XML escape | Example |
|-----------|-----------|---------|
| `<` | `&lt;` | `<l>if (A &lt; B)</l>` |
| `>` | `&gt;` | `<l>if (A &gt; B)</l>` |
| `&` | `&amp;` | `<l>S := 'A' + '&amp;' + 'B'</l>` |

The `par_start` thread syntax uses angle brackets around thread names, which must be escaped:
```xml
<l>par_start&lt;Thread1&gt; : find_data_code_2d (...)</l>
<l>par_start&lt;Thread2&gt; : find_bar_code (...)</l>
<l>par_join ([Thread1,Thread2])</l>
```

### Code lines contain standard HDevelop syntax

```xml
<!-- Assignment -->
<l>Threshold := 128</l>

<!-- Operator call -->
<l>read_image (Image, 'fabrik')</l>
<l>threshold (Image, Region, Threshold, 255)</l>

<!-- Control flow -->
<l>for i := 1 to 10 by 1</l>
<l>    select_obj (Objects, Obj, i)</l>
<l>endfor</l>

<l>while (Condition)</l>
<l>endwhile</l>

<l>if (Area &gt; 1000)</l>
<l>    Result := 'ok'</l>
<l>elseif (Area &gt; 500)</l>
<l>    Result := 'marginal'</l>
<l>else</l>
<l>    Result := 'nok'</l>
<l>endif</l>

<l>switch (Mode)</l>
<l>case 1:</l>
<l>    break</l>
<l>default:</l>
<l>    break</l>
<l>endswitch</l>

<l>try</l>
<l>    read_image (Image, Name)</l>
<l>catch (Exception)</l>
<l>    dev_get_exception_data (Exception, 'error_msg', Msg)</l>
<l>endtry</l>
```

## `<docu>` element (documentation)

Documentation block attached to a procedure. **Required** for every procedure, including `main`.

| Attribute | Value | Description |
|-----------|-------|-------------|
| `id` | string | Must match the parent `<procedure>`'s `name` |

### Multi-language rule

The HDevelop GUI only displays text elements that match the GUI's active language. To ensure descriptions are visible regardless of language setting, provide **two** copies of every text element:

1. **`lang="en_US"`** — English copy
2. **`lang="de_DE"`** — German copy

If the user's locale is known and differs from `en_US`/`de_DE`, add a third copy (e.g., `lang="ja_JP"`).

This pattern is how HALCON's own built-in procedures work (see `HALCONROOT/procedures/general/`). The `<keywords>` and `<chapters>` elements should also be provided in both `en_US` and `de_DE`.

### Documentation child elements

All children are optional, but `<abstract>`, `<short>`, `<parameters>`, and `<example>` should always be present.

#### Core description elements (provide in all three language variants)

| Tag | Description |
|-----|-------------|
| `<abstract>` | Detailed description of what the procedure does. May contain operator refs (`&lt;op:operator_name&gt;`), procedure refs (`&lt;proc:procedure_name&gt;`), and topic refs (`&lt;toc:topic&gt;`) |
| `<short>` | One-line summary |
| `<example>` | Usage example showing how to call the procedure with realistic parameters. Plain text, not XML-escaped |

#### Advisory elements (provide in all three language variants where appropriate)

| Tag | Description |
|-----|-------------|
| `<attention>` | Important notes: required preprocessing, dependency on other procedures, constraints on input data |
| `<warning>` | Things that can go wrong: deprecated features, performance pitfalls, precision limits |
| `<references>` | Academic papers, documentation links, or standards the algorithm is based on |
| `<complexity>` | Computational complexity or performance characteristics |

#### Metadata elements

| Tag | Description |
|-----|-------------|
| `<chapters lang="...">` | Category tags — contains `<item>` elements (e.g., `"Develop"`, `"Inspection"`). Provide in both `en_US` and `de_DE` |
| `<keywords lang="...">` | Search keywords — contains `<item>` elements. Provide in both `en_US` and `de_DE` |
| `<see_also>` | Related procedures/operators — contains `<item>` elements |
| `<alternatives>` | Alternative approaches — contains `<item>` elements |
| `<predecessor>` | Procedures typically called before this one — contains `<item>` elements |
| `<successor>` | Procedures typically called after this one — contains `<item>` elements |
| `<library lang="...">` | Library name (e.g., `"MVTec Standard Procedures"`) |
| `<parameters>` | Parameter documentation — contains `<parameter>` elements |

### `<parameter>` element (documentation)

Documents one procedure parameter in the `<docu>` block.

| Attribute | Value |
|-----------|-------|
| `id` | Parameter name (must match a `<par>` in `<interface>`) |

| Child tag | Description | Example values |
|-----------|-------------|----------------|
| `<default_type>` | Expected type | `"integer"`, `"real"`, `"string"` |
| `<default_value>` | Default value | `"0"`, `"-1"`, `"'true'"` |
| `<description>` | Human-readable description. Provide in **three** variants: no `lang` (fallback), `lang="en_US"`, `lang="de_DE"` | |
| `<mixed_type>` | Accepts multiple types? | `"true"`, `"false"` |
| `<multivalue>` | Accepts tuples? | `"false"`, `"true"`, `"optional"` |
| `<multichannel>` | Multi-channel support? | `"optional"` |
| `<sem_type>` | Semantic type | `"integer"`, `"window"`, `"image"`, `"color"`, `"number"`, `"string"`, `"filename.read"`, `"point.x"` |
| `<type_list>` | Allowed types — contains `<item>` elements | `"integer"`, `"real"`, `"string"` |
| `<values>` | Suggested/allowed values — contains `<item>` elements | `"300"`, `"[300,600]"`, `"'left'"` |

### Full `<docu>` example for a non-main procedure

Note the three-language pattern: a language-independent copy (no `lang`) as fallback, plus `lang="en_US"` and `lang="de_DE"` copies. This matches how HALCON's own built-in procedures are documented.

```xml
<docu id="inspect_threshold_defects">
  <abstract lang="en_US">Inspects a grayscale image for defects using fixed threshold segmentation and area-based filtering. Pixels within the specified gray-value range are segmented, split into connected components, and filtered by area. Returns a pass/fail decision based on whether any qualifying defect regions remain.</abstract>
  <abstract lang="de_DE">Prüft ein Graustufenbild auf Defekte mittels fester Schwellwertsegmentierung und flächenbasierter Filterung. Pixel innerhalb des angegebenen Grauwertbereichs werden segmentiert, in zusammenhängende Komponenten aufgeteilt und nach Fläche gefiltert. Liefert eine Gut/Schlecht-Entscheidung basierend darauf, ob qualifizierende Defektregionen verbleiben.</abstract>
  <short lang="en_US">Threshold-based defect inspection with area filtering.</short>
  <short lang="de_DE">Schwellwertbasierte Defektinspektion mit Flächenfilterung.</short>
  <attention lang="en_US">Input image must be single-channel grayscale. Use rgb1_to_gray for color images.</attention>
  <attention lang="de_DE">Das Eingabebild muss ein einkanaliges Graustufenbild sein. Verwenden Sie rgb1_to_gray für Farbbilder.</attention>
  <chapters lang="en_US">
    <item>Inspection</item>
    <item>Segmentation</item>
  </chapters>
  <chapters lang="de_DE">
    <item>Inspektion</item>
    <item>Segmentierung</item>
  </chapters>
  <example lang="en_US">inspect_threshold_defects (Image, DefectRegions, 0, 120, 100, 999999, JobPass, DefectCount, DefectAreas)</example>
  <example lang="de_DE">inspect_threshold_defects (Image, DefectRegions, 0, 120, 100, 999999, JobPass, DefectCount, DefectAreas)</example>
  <keywords lang="en_US">
    <item>threshold</item>
    <item>defect inspection</item>
    <item>area filter</item>
    <item>pass fail</item>
  </keywords>
  <keywords lang="de_DE">
    <item>Schwellwert</item>
    <item>Defektinspektion</item>
    <item>Flächenfilter</item>
    <item>Gut Schlecht</item>
  </keywords>
  <see_also>
    <item>threshold</item>
    <item>connection</item>
    <item>select_shape</item>
    <item>binary_threshold</item>
  </see_also>
  <parameters>
    <parameter id="Image">
      <default_type>string</default_type>
      <description lang="en_US">Input grayscale image to inspect for defects.</description>
      <description lang="de_DE">Eingabe-Graustufenbild zur Defektinspektion.</description>
      <sem_type>image</sem_type>
      <type_list>
        <item>byte</item>
        <item>uint2</item>
      </type_list>
    </parameter>
    <parameter id="ThresholdMin">
      <default_type>integer</default_type>
      <default_value>0</default_value>
      <description lang="en_US">Minimum gray value for threshold segmentation. Range: 0-255 for byte images.</description>
      <description lang="de_DE">Minimaler Grauwert für die Schwellwertsegmentierung. Bereich: 0-255 für Byte-Bilder.</description>
      <mixed_type>false</mixed_type>
      <multivalue>false</multivalue>
      <sem_type>integer</sem_type>
      <type_list>
        <item>integer</item>
      </type_list>
      <values>
        <item>0</item>
      </values>
    </parameter>
    <parameter id="MinDefectArea">
      <default_type>integer</default_type>
      <default_value>100</default_value>
      <description lang="en_US">Minimum area (in pixels) for a region to be considered a defect. Smaller regions are noise.</description>
      <description lang="de_DE">Mindestfläche (in Pixeln), damit eine Region als Defekt gilt. Kleinere Regionen sind Rauschen.</description>
      <mixed_type>false</mixed_type>
      <multivalue>false</multivalue>
      <sem_type>integer</sem_type>
      <type_list>
        <item>integer</item>
      </type_list>
      <values>
        <item>50</item>
        <item>100</item>
        <item>500</item>
      </values>
    </parameter>
    <parameter id="JobPass">
      <default_type>integer</default_type>
      <description lang="en_US">Pass/fail result. 1 if no defects found (pass), 0 if defects detected (fail).</description>
      <description lang="de_DE">Gut/Schlecht-Ergebnis. 1 wenn keine Defekte gefunden (Gut), 0 wenn Defekte erkannt (Schlecht).</description>
      <sem_type>integer</sem_type>
      <values>
        <item>0</item>
        <item>1</item>
      </values>
    </parameter>
  </parameters>
</docu>
```

### `<docu>` example for `main`

The `main` procedure should also be documented. Since `main` has no interface parameters, use `<parameters/>`. The `<abstract>` should describe the script's purpose and assumptions.

```xml
<docu id="main">
  <abstract lang="en_US">Unit test for threshold-based defect inspection. Reads a test image, runs the inspection procedure, and displays the result. All parameters are global variables that can be overridden via hrun -D.</abstract>
  <abstract lang="de_DE">Unit-Test für schwellwertbasierte Defektinspektion. Liest ein Testbild, führt die Inspektionsprozedur aus und zeigt das Ergebnis an. Alle Parameter sind globale Variablen, die über hrun -D überschrieben werden können.</abstract>
  <short lang="en_US">Test script for threshold defect inspection.</short>
  <short lang="de_DE">Testskript für Schwellwert-Defektinspektion.</short>
  <parameters/>
</docu>
```

## Multi-procedure files

### `.hdev` scripts with local procedures

A `.hdev` script can contain `main` plus helper procedures. All procedures share the same `<hdevelop>` root. **Both `main` and helper procedures must have `<docu>` blocks** with all three language variants:

```xml
<hdevelop file_version="1.2" halcon_version="25.11.0.0">

  <procedure name="main">
    <interface/>
    <body>
      <l>read_image (Image, 'fabrik')</l>
      <l>helper_proc (Image, Result)</l>
    </body>
    <docu id="main">
      <abstract lang="en_US">Demonstrates helper_proc by computing the area of a test image.</abstract>
      <abstract lang="de_DE">Demonstriert helper_proc durch Berechnung der Fläche eines Testbildes.</abstract>
      <short>Demo for helper_proc.</short>
      <short lang="en_US">Demo for helper_proc.</short>
      <short lang="de_DE">Demo für helper_proc.</short>
      <parameters/>
    </docu>
  </procedure>

  <procedure name="helper_proc">
    <interface>
      <io><par name="Image" base_type="iconic" dimension="0"/></io>
      <oc><par name="Result" base_type="ctrl" dimension="0"/></oc>
    </interface>
    <body>
      <l>area_center (Image, Result, Row, Column)</l>
      <l>return ()</l>
    </body>
    <docu id="helper_proc">
      <abstract lang="en_US">Computes the area of the image domain.</abstract>
      <abstract lang="de_DE">Berechnet die Fläche des Bildbereichs.</abstract>
      <short lang="en_US">Compute image domain area.</short>
      <short lang="de_DE">Bildbereichsfläche berechnen.</short>
      <example lang="en_US">helper_proc (Image, Area)</example>
      <example lang="de_DE">helper_proc (Image, Area)</example>
      <parameters>
        <parameter id="Image">
          <description lang="en_US">Input image.</description>
          <description lang="de_DE">Eingabebild.</description>
          <sem_type>image</sem_type>
        </parameter>
        <parameter id="Result">
          <description lang="en_US">Area of the image domain in pixels.</description>
          <description lang="de_DE">Fläche des Bildbereichs in Pixeln.</description>
          <default_type>integer</default_type>
          <sem_type>integer</sem_type>
        </parameter>
      </parameters>
    </docu>
  </procedure>

</hdevelop>
```

### `.hdpl` procedure libraries

Libraries are marked with `<library/>` and can use `access="local"` to hide internal helpers:

```xml
<hdevelop file_version="1.2" halcon_version="25.11.0.0">
<library/>

<!-- Internal helper: not visible to importers -->
<procedure name="internal_helper" access="local">
  <interface>
    <ic><par name="Input" base_type="ctrl" dimension="0"/></ic>
    <oc><par name="Output" base_type="ctrl" dimension="0"/></oc>
  </interface>
  <body>
    <l>Output := Input * 2</l>
    <l>return ()</l>
  </body>
</procedure>

<!-- Public procedure: visible to importers -->
<procedure name="public_api">
  <interface>
    <ic><par name="Value" base_type="ctrl" dimension="0"/></ic>
    <oc><par name="Result" base_type="ctrl" dimension="0"/></oc>
  </interface>
  <body>
    <l>internal_helper (Value, Result)</l>
    <l>return ()</l>
  </body>
</procedure>

</hdevelop>
```

For the `hrun` CLI reference (help output, usage patterns, authoring guidelines), see [`hrun_reference.md`](hrun_reference.md).
