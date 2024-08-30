Enhancing VLSI Design Efficiency: Tackling Congestion and Shorts with Practical Approaches and PnR Tool (ICC2)
Overview
This repository provides comprehensive insights and practical approaches to manage and mitigate congestion and shorts in VLSI design, with a focus on using the Place and Route (PnR) tool ICC2. The methods and techniques discussed here are essential for achieving efficient and reliable integrated circuits, especially as designs become more complex and densely packed.

Author: Ishu Shukla, eInfochips, an Arrow company

Table of Contents
Abstract
Congestion in VLSI Design
Understanding Congestion
Causes of Congestion
Congestion Alleviation Techniques
PnR Tool Commands for Congestion
Shorts in VLSI Design
Understanding Shorts
Mitigation Techniques
PnR Tool Commands for Shorts
Conclusion
References
Abstract
Effective management of congestion is crucial for the efficient and reliable operation of modern integrated circuits, which are becoming increasingly complex and densely packed with millions of transistors. This repository illustrates congestion, shorts, and practical approaches to fixing both issues at lower and higher technology nodes. Additionally, it includes PnR tool (ICC2) related commands and their uses to overcome these issues.

Congestion in VLSI Design
Understanding Congestion
Congestion in VLSI (Very Large-Scale Integration) design occurs when the number of routing tracks is insufficient for the required routing. This issue becomes more prevalent as designs grow in complexity, requiring careful optimization to meet timing, power, and area constraints.


Figure 1: Congestion hotspots highlighted in red.

Congestion Report
Global Routing Cells (GRC): Small squares into which the core area is divided during initial placement.
Overflow Total: The total number of overflow routes across all GRCs.
Overflow Max: The maximum number of overflow routes in a single GRC.
Overflow Percentage (%): Indicates the percentage of GRCs with routing overflow.
Note: GRC overflow should be less than 1% to avoid significant routing and DRC violations, which can degrade timing due to increased wire length, capacitance, and resistance.

Causes of Congestion
Bad floorplanning/inappropriate placement of macros
High standard cell density in specific areas
High proximity of standard cells to macros
Routing blockages over standard cells

Figure 2: Routing blockages over standard cells.

High port density

Figure 3: High port density areas.

Restricted scan chain reordering and mixing/swapping
Improper netlist optimization during synthesis

Figure 4: Standard cells close to macros causing congestion.

Congestion Alleviation Techniques
Placement Blockages: Spread standard cells in highly dense areas or restrict placement in areas with limited routing resources.


Figure 5: Partial placement blockage example.


Figure 6: Hard placement blockage example.

tcl
Copy code
create_placement_blockage -type partial -blocked_percentage 50 -boundary {{llx lly} {urx ury}} -name Partial_PB
create_placement_blockage -type hard -boundary {{lrx lry} {urx ury}} -name Hard_PB
Cell Padding: Apply keepout margins around high-pin-count cells to reduce congestion.


Figure 7: Keepout margin around a macro.

tcl
Copy code
create_keepout_margin -type hard -outer {5 5 5 5 } [get_cells *macro_name*]

Figure 8: Cell padding technique.

tcl
Copy code
create_keepout_margin -type hard -outer {3.9200 3.9200 3.9200 3.9200} [get_cells cell_name]
Keepout Margin/HALO: Create keepout margins around macros to avoid congestion and net detouring.

Modify PG Grid: Reduce the number or width of PG stripes to increase routing resources (with consideration of EM and IR drop trade-offs).

Congestion-Driven Placement: Adjust placement strategies to minimize congestion.

Topographic Synthesis: Use physical constraints during synthesis for better netlist optimization.

SPG and Non-SPG Placement: Experiment with SPG and non-SPG techniques for congestion relief.

tcl
Copy code
set_app_options -name place_opt.flow.do_spg -value true
read_def -add_def_only_objects {cells} -convert_sites <def file>
place_opt
Refine Placement: Perform incremental optimization after detailed placement.

tcl
Copy code
refine_placement -effort high -congestion_effort high
Shorts in VLSI Design
Understanding Shorts
Shorts occur when two different nets intersect or touch each other in the same metal layer. This can cause significant issues in the circuit's functionality and reliability.


Figure 9: Example of a short between two nets.

Mitigation Techniques
Manually Adjusting Nets: Shift and reroute shorted nets manually for small-scale shorts.


Figure 10: Corrected short by shifting the net.

Deleting and Rerouting Shorted Nets: Use the PnR tool to remove and reroute shorted nets.

tcl
Copy code
remove_routes -detail_route -global_route -shield_route -nets “$net_name”
route_eco -open_net_driven true.
Running Detail Routing: Perform iterative detail routing to minimize shorts and DRC violations.

tcl
Copy code
route_detail –max_number_iterations 5
Using Routing Blockages: Place blockages during floorplanning and remove them after initial routing to mitigate shorts in specific areas.


Figure 11: Routing blockage placement at a corner.

tcl
Copy code
remove_routing_blockages *corner_blockages*
route_detail -incremental true
Applying Routing Guides: Limit routing in specific layers by using routing guides during placement.

tcl
Copy code
create_routing_guide -layers METAL2 -vertical_track_utilization 70 -boundary $bbox -name rg1
Conclusion
By implementing the practical approaches and commands detailed in this repository, designers can significantly improve the efficiency and reliability of VLSI designs. These techniques are essential for managing congestion and shorts in the increasingly complex landscape of semiconductor design.
