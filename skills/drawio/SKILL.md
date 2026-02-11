---
name: drawio
description: "Use this skill whenever the user wants to create, edit, or manipulate Draw.io diagrams (.drawio, .drawio.svg, .drawio.png, .dio files). Triggers include: any mention of 'draw.io', 'drawio', 'diagram', 'flowchart', 'orgchart', 'architecture diagram', 'sequence diagram', 'network diagram', 'ER diagram', 'UML', 'state machine', 'mind map', or requests to visually represent processes, systems, or relationships. Also use when the user asks to convert text descriptions into visual diagrams, or to modify existing .drawio files. Do NOT use for Mermaid diagrams (.mermaid), SVG hand-coding, image generation, or non-diagram visualizations like charts/graphs (use xlsx or React for those)."
---

# Draw.io Diagram Creation and Editing

## Overview

A `.drawio` file is an XML file based on the **mxGraph** format. It can be opened and edited in VS Code with the Draw.io extension (hediet.vscode-drawio). Files can also use `.dio`, `.drawio.svg`, or `.drawio.png` extensions.

## Quick Reference

| Task                  | Approach                                                                         |
| --------------------- | -------------------------------------------------------------------------------- |
| Create new diagram    | Write raw XML in `.drawio` format — see structure below                          |
| Edit existing diagram | Parse XML → modify nodes/edges → write back                                      |
| Multi-page diagram    | Add multiple `<diagram>` elements inside `<mxfile>`                              |
| Preferred format      | `.drawio` for version control (clean diffs), `.drawio.svg` for embedding in docs |

---

## XML Structure

Every `.drawio` file follows this skeleton:

```xml
<mxfile host="Claude" modified="2025-01-01T00:00:00.000Z" agent="Claude" version="24.0.0" type="device">
  <diagram name="Page-1" id="page1">
    <mxGraphModel dx="1422" dy="762" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1169" pageHeight="827" math="0" shadow="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
        <!-- All user cells go here, with parent="1" -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

### Critical Rules

1. **Cell id="0"** is the invisible root. It MUST exist. Never skip it.
2. **Cell id="1"** is the default layer, with `parent="0"`. It MUST exist.
3. **All visible cells** (shapes, edges, labels) must have `parent="1"` (or another layer/group cell).
4. **IDs must be unique** across the entire diagram. Use a consistent scheme like `node-1`, `edge-1`, `group-1`, etc.
5. **Coordinates**: `x` increases rightward, `y` increases downward. Origin (0,0) is the top-left of the canvas.
6. **pageWidth/pageHeight**: Use `1169x827` for landscape A4, `827x1169` for portrait A4. Adjust `dx`/`dy` for canvas scroll offset.

---

## Creating Shapes (Vertices)

A shape is an `mxCell` with a `vertex="1"` attribute and an `mxGeometry` child:

```xml
<mxCell id="node-1" value="My Label" style="rounded=1;whiteSpace=wrap;html=1;" vertex="1" parent="1">
  <mxGeometry x="100" y="100" width="120" height="60" as="geometry" />
</mxCell>
```

### Common Shape Styles

| Shape                | Style String                                                                               |
| -------------------- | ------------------------------------------------------------------------------------------ |
| Rectangle            | `rounded=0;whiteSpace=wrap;html=1;`                                                        |
| Rounded rectangle    | `rounded=1;whiteSpace=wrap;html=1;`                                                        |
| Circle / Ellipse     | `ellipse;whiteSpace=wrap;html=1;`                                                          |
| Diamond (decision)   | `rhombus;whiteSpace=wrap;html=1;`                                                          |
| Cylinder (database)  | `shape=cylinder3;whiteSpace=wrap;html=1;boundedLbl=1;backgroundOutline=1;size=15;`         |
| Document             | `shape=document;whiteSpace=wrap;html=1;boundedLbl=1;`                                      |
| Parallelogram (I/O)  | `shape=parallelogram;perimeter=parallelogramPerimeter;whiteSpace=wrap;html=1;fixedSize=1;` |
| Cloud                | `ellipse;shape=cloud;whiteSpace=wrap;html=1;`                                              |
| Hexagon              | `shape=hexagon;perimeter=hexagonPerimeter2;whiteSpace=wrap;html=1;fixedSize=1;`            |
| Actor (stick figure) | `shape=umlActor;verticalLabelPosition=bottom;verticalAlign=top;html=1;`                    |
| Note / Callout       | `shape=note;whiteSpace=wrap;html=1;backgroundOutline=1;size=15;`                           |
| Rounded process      | `rounded=1;whiteSpace=wrap;html=1;arcSize=50;`                                             |
| Start/End (stadium)  | `rounded=1;whiteSpace=wrap;html=1;arcSize=50;` (with small height)                         |
| Manual input         | `shape=manualInput;whiteSpace=wrap;html=1;`                                                |
| Container / Group    | `rounded=0;whiteSpace=wrap;html=1;container=1;collapsible=0;`                              |

### Style Properties Reference

| Property        | Values                                            | Description                            |
| --------------- | ------------------------------------------------- | -------------------------------------- |
| `fillColor`     | `#dae8fc`, `#d5e8d4`, `none`, ...                 | Background color (hex)                 |
| `strokeColor`   | `#6c8ebf`, `#82b366`, `none`, ...                 | Border color (hex)                     |
| `fontColor`     | `#333333`, `#FFFFFF`, ...                         | Text color                             |
| `fontSize`      | `12`, `14`, `18`, ...                             | Font size in points                    |
| `fontStyle`     | `0`=normal, `1`=bold, `2`=italic, `3`=bold+italic | Font style bitmask                     |
| `strokeWidth`   | `1`, `2`, `3`, ...                                | Border thickness                       |
| `dashed`        | `0`, `1`                                          | Dashed border                          |
| `opacity`       | `0`–`100`                                         | Cell opacity                           |
| `shadow`        | `0`, `1`                                          | Drop shadow                            |
| `align`         | `left`, `center`, `right`                         | Horizontal text alignment              |
| `verticalAlign` | `top`, `middle`, `bottom`                         | Vertical text alignment                |
| `whiteSpace`    | `wrap`                                            | Enable text wrapping (always include)  |
| `html`          | `1`                                               | Enable HTML in labels (always include) |
| `container`     | `1`                                               | Makes cell a container for child cells |
| `collapsible`   | `0`, `1`                                          | Whether container can collapse         |

### Recommended Color Palettes

**Blue theme** (default Draw.io):

- Fill: `#dae8fc` / Stroke: `#6c8ebf`

**Green theme** (success, OK):

- Fill: `#d5e8d4` / Stroke: `#82b366`

**Orange theme** (warning):

- Fill: `#fff2cc` / Stroke: `#d6b656`

**Red theme** (error, danger):

- Fill: `#f8cecc` / Stroke: `#b85450`

**Purple theme**:

- Fill: `#e1d5e7` / Stroke: `#9673a6`

**Gray theme** (neutral):

- Fill: `#f5f5f5` / Stroke: `#666666`

**Dark / header**:

- Fill: `#333333` / Stroke: `#000000` / Font: `#FFFFFF`

---

## Creating Connections (Edges)

An edge is an `mxCell` with `edge="1"`, connecting a `source` to a `target`:

```xml
<mxCell id="edge-1" value="" style="edgeStyle=orthogonalEdgeStyle;rounded=0;" edge="1" source="node-1" target="node-2" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

### Edge Style Options

| Style                                | Description                          |
| ------------------------------------ | ------------------------------------ |
| `edgeStyle=orthogonalEdgeStyle;`     | Right-angle connectors (most common) |
| `edgeStyle=elbowEdgeStyle;`          | Single elbow connector               |
| `edgeStyle=entityRelationEdgeStyle;` | ER-diagram style                     |
| `curved=1;`                          | Curved lines                         |
| (no edgeStyle)                       | Straight line                        |

### Arrow Styles

| Property                | Values                                 |
| ----------------------- | -------------------------------------- |
| `endArrow=classic;`     | Standard arrowhead (default)           |
| `endArrow=block;`       | Filled triangle                        |
| `endArrow=open;`        | Open arrowhead                         |
| `endArrow=diamond;`     | Diamond (aggregation)                  |
| `endArrow=diamondThin;` | Thin diamond (composition when filled) |
| `endArrow=none;`        | No arrow                               |
| `startArrow=...`        | Same options, for start of edge        |
| `endFill=0;`            | Hollow end arrow                       |
| `endFill=1;`            | Filled end arrow                       |
| `startFill=0;`          | Hollow start arrow                     |

### Edge Labels

Add a label to an edge via `value="label text"`, or with a child label cell:

```xml
<mxCell id="edge-1" value="yes" style="edgeStyle=orthogonalEdgeStyle;" edge="1" source="node-1" target="node-2" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

---

## Groups and Containers

To group cells visually (e.g., a swimlane or subsystem box):

```xml
<!-- The container -->
<mxCell id="group-1" value="My Group" style="rounded=0;whiteSpace=wrap;html=1;container=1;collapsible=0;fillColor=#f5f5f5;strokeColor=#666666;verticalAlign=top;fontStyle=1;fontSize=14;" vertex="1" parent="1">
  <mxGeometry x="50" y="50" width="300" height="200" as="geometry" />
</mxCell>

<!-- Child cells with parent="group-1" — coordinates are relative to the group -->
<mxCell id="node-inside" value="Child" style="rounded=1;whiteSpace=wrap;html=1;" vertex="1" parent="group-1">
  <mxGeometry x="20" y="40" width="100" height="40" as="geometry" />
</mxCell>
```

**Key**: child cells use `parent="group-1"` and their x/y are relative to the group's top-left corner.

---

## Swimlanes

Horizontal swimlane:

```xml
<mxCell id="lane-1" value="Department A" style="shape=swimlane;startSize=30;horizontal=1;fillColor=#dae8fc;strokeColor=#6c8ebf;collapsible=0;fontStyle=1;" vertex="1" parent="1">
  <mxGeometry x="50" y="50" width="600" height="200" as="geometry" />
</mxCell>
```

Vertical swimlane: add `horizontal=0;` to the style.

---

## Layout Best Practices

### Spacing and Alignment

- **Minimum node width**: 120px for readability.
- **Minimum node height**: 40–60px.
- **Horizontal spacing** between nodes: 40–60px.
- **Vertical spacing** between rows: 60–80px.
- **Align nodes on a grid**: use multiples of 10 or 20 for x/y coordinates.

### Flowchart Layout (Top to Bottom)

```
Row 0 (y=40):    [Start]
Row 1 (y=140):   [Process A]    [Process B]
Row 2 (y=240):   [Decision]
Row 3 (y=340):   [End]
```

Use consistent x-spacing: e.g., center column at x=200, left at x=50, right at x=350.

### Left-to-Right Layout

For process flows and pipelines, lay out horizontally:

- Column 0: x=40
- Column 1: x=220 (120 width + 100 gap)
- Column 2: x=400

### General Tips

- Use `orthogonalEdgeStyle` for clean right-angle connectors.
- Color-code by category (e.g., blue for services, green for databases, orange for users).
- Always include `whiteSpace=wrap;html=1;` on shapes so labels wrap properly.
- For decision diamonds, make them at least 120x80 so the text fits.
- Put labels on edges when the relationship isn't obvious.

---

## Common Diagram Patterns

### Flowchart

```xml
<mxfile host="Claude" modified="2025-01-01T00:00:00.000Z" agent="Claude" version="24.0.0" type="device">
  <diagram name="Flowchart" id="flowchart">
    <mxGraphModel dx="1422" dy="762" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1169" pageHeight="827" math="0" shadow="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
        <mxCell id="start" value="Start" style="rounded=1;whiteSpace=wrap;html=1;arcSize=50;fillColor=#d5e8d4;strokeColor=#82b366;" vertex="1" parent="1">
          <mxGeometry x="200" y="40" width="120" height="40" as="geometry" />
        </mxCell>
        <mxCell id="process1" value="Process Step" style="rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;" vertex="1" parent="1">
          <mxGeometry x="200" y="130" width="120" height="60" as="geometry" />
        </mxCell>
        <mxCell id="decision1" value="Condition?" style="rhombus;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;" vertex="1" parent="1">
          <mxGeometry x="195" y="240" width="130" height="80" as="geometry" />
        </mxCell>
        <mxCell id="end" value="End" style="rounded=1;whiteSpace=wrap;html=1;arcSize=50;fillColor=#f8cecc;strokeColor=#b85450;" vertex="1" parent="1">
          <mxGeometry x="200" y="380" width="120" height="40" as="geometry" />
        </mxCell>
        <mxCell id="e1" style="edgeStyle=orthogonalEdgeStyle;" edge="1" source="start" target="process1" parent="1">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="e2" style="edgeStyle=orthogonalEdgeStyle;" edge="1" source="process1" target="decision1" parent="1">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="e3" value="Yes" style="edgeStyle=orthogonalEdgeStyle;" edge="1" source="decision1" target="end" parent="1">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

### Architecture Diagram Pattern

For system/architecture diagrams:

- Use containers (`container=1`) for logical groupings (e.g., "Cloud", "On-Premise").
- Use cylinders for databases, clouds for external services, rectangles for services.
- Use dashed edges (`dashed=1;`) for async/optional connections.
- Add a title with a large text-only cell at the top: `style="text;html=1;fontSize=20;fontStyle=1;align=center;"`.

### UML Class Diagram Cells

Use HTML labels for multi-section boxes:

```xml
<mxCell id="class-1" value="&lt;b&gt;ClassName&lt;/b&gt;&lt;hr&gt;- field1: String&lt;br&gt;- field2: int&lt;hr&gt;+ method1()&lt;br&gt;+ method2()" style="rounded=0;whiteSpace=wrap;html=1;align=left;overflow=fill;verticalAlign=top;" vertex="1" parent="1">
  <mxGeometry x="100" y="100" width="200" height="120" as="geometry" />
</mxCell>
```

**Note**: In XML, HTML inside `value` must be entity-escaped: `<` → `&lt;`, `>` → `&gt;`.

---

## HTML Labels

Draw.io supports HTML in cell values when `html=1` is in the style. You MUST entity-escape all HTML:

| Character | Entity   |
| --------- | -------- |
| `<`       | `&lt;`   |
| `>`       | `&gt;`   |
| `&`       | `&amp;`  |
| `"`       | `&quot;` |

Example: bold + line break:

```xml
value="&lt;b&gt;Title&lt;/b&gt;&lt;br&gt;Subtitle"
```

---

## Multi-Page Diagrams

Simply add more `<diagram>` elements:

```xml
<mxfile host="Claude">
  <diagram name="Overview" id="page1">
    <mxGraphModel ...>
      <root>...</root>
    </mxGraphModel>
  </diagram>
  <diagram name="Details" id="page2">
    <mxGraphModel ...>
      <root>...</root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

---

## Editing Existing Diagrams

To modify a `.drawio` file:

1. Read the XML file content.
2. Parse with any XML parser or string manipulation.
3. Add/remove/modify `mxCell` elements.
4. Write the modified XML back.

**Important**: preserve the existing `id` values to avoid breaking edge connections. When adding new cells, use IDs that don't conflict with existing ones.

---

## Validation Checklist

Before finalizing a `.drawio` file, verify:

- [ ] `mxCell id="0"` and `mxCell id="1" parent="0"` are present
- [ ] All cell IDs are unique
- [ ] All `vertex` cells have `<mxGeometry>` with x, y, width, height
- [ ] All `edge` cells reference valid `source` and `target` IDs
- [ ] All edge geometries have `relative="1"`
- [ ] HTML in `value` attributes is entity-escaped (`&lt;`, `&gt;`, `&amp;`)
- [ ] `whiteSpace=wrap;html=1;` is present on all shape styles
- [ ] No overlapping nodes (check x/y/width/height don't clash)
- [ ] Child cells of groups use the group ID as `parent`, not `"1"`
- [ ] The XML is well-formed (valid XML, all tags closed)

---

## Output

- Always write the file with `.drawio` extension (e.g., `diagram.drawio`).
- Use UTF-8 encoding.
- The file must be valid XML — test with `xmllint --noout file.drawio` if available.
- Place the final file in the output directory so the user can download and open it in VS Code with the Draw.io extension.
