# Enhancing VLSI Design Efficiency: Tackling Congestion and Shorts with Practical Approaches and PnR Tool (ICC2)

## Abstract

As integrated circuits (ICs) continue to evolve, the complexity and density of these systems increase significantly, leading to challenges in design, particularly in terms of congestion and shorts. Effective management of these issues is crucial to ensure the efficient and reliable operation of modern ICs. This paper provides an in-depth exploration of congestion, shorts, and the practical approaches required to mitigate these problems, especially at lower and higher technology nodes. Additionally, it delves into the use of PnR (Place and Route) tools, specifically ICC2, to overcome these challenges with a focus on real-world applications and commands.

## Table of Contents
1. [Introduction](#introduction)
2. [Congestion in VLSI Design](#congestion-in-vlsi-design)
    - [Understanding Congestion](#understanding-congestion)
    - [Congestion Report Analysis](#congestion-report-analysis)
    - [Causes of Congestion](#causes-of-congestion)
    - [Techniques for Congestion Alleviation](#techniques-for-congestion-alleviation)
3. [Shorts in VLSI Design](#shorts-in-vlsi-design)
    - [Understanding Shorts](#understanding-shorts)
    - [Mitigation of Shorts](#mitigation-of-shorts)
4. [Practical ICC2 Commands](#practical-icc2-commands)
6. [Conclusion](#conclusion)
7. [References](#references)

## Introduction

The VLSI (Very Large Scale Integration) design landscape has seen an exponential increase in the number of transistors per chip, which has led to significant design complexities. Among these challenges, congestion and shorts are two of the most critical issues that designers must address to ensure the functionality and performance of ICs. Congestion, which occurs when there are insufficient routing resources to accommodate all the required connections, can lead to severe timing and power issues. Similarly, shorts, which happen when different nets accidentally connect at the same layer, can cause catastrophic failures if not properly mitigated.

This document aims to provide a comprehensive guide to understanding, analyzing, and mitigating congestion and shorts using practical approaches and the ICC2 PnR tool. The paper also includes a series of commands specific to ICC2 that are designed to assist in resolving these issues efficiently.

## Congestion in VLSI Design

### Understanding Congestion

Congestion in VLSI design refers to the scenario where the available routing resources are insufficient to connect all the necessary wires between the different components of the design. As the complexity of IC designs increases, managing congestion becomes a significant challenge that requires careful consideration to ensure that the design meets timing, power, and area constraints. Congestion hotspots are typically highlighted by PnR tools like ICC2 as red areas on the design, indicating regions where routing resources are particularly scarce.

![Congestion Hotspots](path/to/your/image1.png)

*Figure 1: Congestion hotspots in the design as indicated by red areas.*

### Congestion Report Analysis

In ICC2, congestion analysis is a critical step in understanding the severity and location of congestion in the design. The congestion report provides several key metrics, including:

- **Global Routing Cells (GRCs)**: The core area of the chip is divided into equally sized small squares called GRCs during the initial placement phase.
- **Overflow Total**: This is the total number of overflow routes across all GRCs, indicating how many additional routing tracks are needed beyond what is available.
- **Overflow Max**: Represents the maximum overflow in a single GRC, considering both horizontal and vertical routing directions.
- **Overflow Percentage (%)**: Indicates the percentage of GRCs that have an overflow. A high overflow percentage is a strong indicator of severe congestion, which can lead to routing challenges and timing violations.

![Congestion Report](path/to/your/image2.png)

*Figure 2: Example of a congestion report showing overflow metrics.*

For example, if the GRC overflow percentage exceeds 1%, routing may become difficult, leading to complex shorts and Design Rule Check (DRC) violations. These issues can also cause timing degradation by increasing the length of interconnects, which in turn increases the capacitance and resistance of the wires, slowing down signal propagation.

### Causes of Congestion

Several factors can contribute to congestion in VLSI design, including:

1. **Bad Floorplan/Inappropriate Placement of Macros**: Poorly planned floorplans can lead to congestion hotspots, particularly around large macros.
2. **High Standard Cell Density**: When too many standard cells are placed in close proximity, the demand for routing resources increases, leading to congestion.
3. **High Number of Standard Cells Near Macros**: Proximity to macros can exacerbate congestion due to the increased routing requirements.
4. **Routing Blockages**: Blockages placed over standard cells can restrict available routing paths, contributing to congestion.
5. **High Port Density**: Areas with a high density of ports can become congested due to the large number of connections required.
6. **Scan Chain Reordering Restrictions**: Limited reordering options can lead to suboptimal routing paths, increasing congestion.
7. **Improper Netlist Optimization During Synthesis**: Poorly optimized netlists can result in routing inefficiencies that exacerbate congestion.

### Techniques for Congestion Alleviation

To address congestion, several techniques can be employed:

1. **Placement Blockages**: By creating partial or hard placement blockages, designers can control the density of standard cells in congested areas.
    - **Partial Placement Blockage**: Used to spread standard cells more evenly across the design by blocking a percentage of placement in a given area.
      ```shell
      create_placement_blockage -type partial -blocked_percentage 50 -boundary {{llx lly} {urx ury}} -name Partial_PB
      ```
    - **Hard Placement Blockage**: Restricts placement entirely within a specific region to prevent congestion.
      ```shell
      create_placement_blockage -type hard -boundary { {lrx lry} {urx ury} } –name Hard_PB
      ```

    ![Partial Placement Blockage](path/to/your/image3.png)
    *Figure 3: Partial placement blockage example.*

    ![Hard Placement Blockage](path/to/your/image4.png)
    *Figure 4: Hard placement blockage restricting cell placement.*

2. **Keep Out Margin/Halo**: A keepout margin around a macro ensures that no other cells are placed too close, reducing congestion and net detouring.

    ![Keepout Margin](path/to/your/image5.png)
    *Figure 5: Keepout margin (Halo) around a macro to prevent congestion.*

    ```shell
    create_keepout_margin -type hard -outer {5 5 5 5 } [get_cells *macro_name*]
    ```

3. **Cell Padding**: By applying a keepout margin around high-pin-count standard cells, congestion can be mitigated in areas with high routing demand.

    ![Cell Padding](path/to/your/image6.png)
    *Figure 6: Example of cell padding to manage routing congestion.*

    ```shell
    create_keepout_margin -type hard -outer {3.9200 3.9200 3.9200 3.9200} [get_cells cell_name]
    ```

4. **Modify PG Grid**: Reducing the number or width of power/ground (PG) stripes can increase available routing resources, though this must be balanced against potential electromagnetic (EM) and IR drop issues.

5. **Topographical Synthesis**: By incorporating physical design data early in the synthesis process, topographical synthesis can help produce a more congestion-aware netlist.

6. **SPG and Non-SPG Placement**: Different placement strategies can be explored to find the best approach for minimizing congestion.
    - **SPG Placement**: Uses DEF data for coarse placement.
      ```shell
      set_app_options -name place_opt.flow.do_spg -value true
      ```
    - **Non-SPG Placement**: Lets the PnR tool determine placement without DEF data.

7. **Refine Placement**: Incremental placement refinement can help optimize for congestion after initial placement has been completed.
    ```shell
    refine_placement -effort high -congestion_effort high
    ```

## Shorts in VLSI Design

### Understanding Shorts

Shorts in VLSI design occur when the shapes of two different nets intersect or touch each other on the same metal layer. These shorts can lead to functional failures if not corrected. They are typically identified during the routing or LVS (Layout Versus Schematic) checking stages.

![Shorts Example](path/to/your/image7.png)

*Figure 7: Example of a short where two nets intersect on the same metal layer.*

### Mitigation of Shorts

To mitigate shorts, the following approaches can be employed:

1. **Manual Fixes for Low Counts of Shorts**: If the design has a small number of shorts (single or double digits), they can often be fixed manually by adjusting the routing.
    - **Shift Nets**: Manually adjust the routing of the shorted nets to eliminate overlaps.

    ![Short Fix by Shifting Nets](path/to/your/image8.png)
    *Figure 8: Shifting the red net to eliminate the short.*

2. **ECO Routing**: For larger numbers of shorts, an Engineering Change Order (ECO) approach can be used to reroute the shorted nets.
    - **Remove Shorted Nets**:
      ```shell
      Remove_routes -detail_route -global_route -shield_route -nets "$net_name"
      ```
    - **Reroute Nets**:
      ```shell
      route_eco -open_net_driven true
      ```

3. **Detail Routing with Multiple Iterations**: Running multiple iterations of detailed routing can help to systematically eliminate shorts.
    ```shell
    route_detail –max_number_iterations 5
    ```

4. **Use Routing Blockages**: In cases where shorts occur at specific locations, such as corners, placing routing blockages can prevent shorts during the initial routing phase.

    ![Routing Blockage at Corner](path/to/your/image9.png)
    *Figure 9: Adding a routing blockage at the corner to prevent shorts.*

    - **Add Routing Blockages**:
      ```shell
      remove_routing_blockages *corner_blockages*
      route_detail -incremental true
      ```

5. **Apply Routing Guides**: To limit the number of routing tracks used in a specific metal layer, routing guides can be applied.
    - **Create Routing Guide**:
      ```shell
      Set bbox {{ 1125.8800 -210.1300} { 1404.2000 747.9400}}
      create_routing_guide -layers METAL2 -vertical_track_utilization 70 -boundary $bbox -name rg1
      ```

## Practical ICC2 Commands

This section provides a summary of the key ICC2 commands used throughout the paper for tackling congestion and shorts. These commands are essential for PnR engineers to manage and mitigate design challenges effectively.

- **Placement Blockages**:
  - `create_placement_blockage -type partial -blocked_percentage 50 -boundary {{llx lly} {urx ury}} -name Partial_PB`
  - `create_placement_blockage -type hard -boundary { {lrx lry} {urx ury} } –name Hard_PB`

- **Keep Out Margin/Halo**:
  - `create_keepout_margin -type hard -outer {5 5 5 5 } [get_cells *macro_name*]`

- **Cell Padding**:
  - `create_keepout_margin -type hard -outer {3.9200 3.9200 3.9200 3.9200} [get_cells cell_name]`

- **Modify PG Grid**: Adjust PG grid settings to balance routing resources and power integrity.

- **SPG and Non-SPG Placement**:
  - `set_app_options -name place_opt.flow.do_spg -value true`
  - `read_def -add_def_only_objects {cells} -convert_sites <def file>`

- **Refine Placement**:
  - `refine_placement -effort high -congestion_effort high`

- **ECO Routing**:
  - `Remove_routes -detail_route -global_route -shield_route -nets "$net_name"`
  - `route_eco -open_net_driven true`

- **Detail Routing**:
  - `route_detail –max_number_iterations 5`

- **Routing Guides**:
  - `create_routing_guide -layers METAL2 -vertical_track_utilization 70 -boundary $bbox -name rg1`

## Conclusion

This paper provides a comprehensive guide to managing and mitigating congestion and shorts in VLSI design. By leveraging the practical approaches and ICC2 commands outlined, designers can enhance the efficiency and reliability of their designs, particularly in the face of increasing complexity and density in modern ICs. The techniques discussed are essential tools for any VLSI designer aiming to optimize their design for performance, power, and area while minimizing potential issues that could compromise the final product.

## References

For more detailed explanations and the complete set of figures, please refer to the full paper and the references therein.
