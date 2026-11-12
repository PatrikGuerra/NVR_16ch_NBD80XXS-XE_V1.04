# Component height estimates for enclosure CAD

Recorded 2026-11-11. Units: mm. See [MECHANICAL.md](MECHANICAL.md) for X/Y coordinates and board-variant limitations.

## Datum and evidence

Z=0 is the top/component-side PCB surface. Height means the highest point of the installed, **unmated** component above that surface; solder tails below the PCB are excluded. Do not use these heights as socket centerline elevations or panel-opening dimensions.

The repository [front photograph](images/frontBoard.jpg) and the [JFTech PDF photo and diagram](https://openipc.org/board-files/xiongmai-nbd88x10s-kl-v3-jftech/NBD88X10S-KL-V3.pdf) identify connector families. They do not reveal reliable exact manufacturer part numbers. The oblique photo is not calibrated in Z, so planar X/Y image scale cannot be used to measure height. Manufacturer dimensions below belong to **comparison parts**, not confirmed components on this board. Confidence refers to the family/shape comparison, not measurement accuracy. Ranges are engineering estimates, not guaranteed bounds.

## Heights and preliminary CAD reservations

"Reserve" is an intentionally rounded, preliminary top-of-component keepout measured from Z=0. It includes a small geometric allowance beyond the estimate; it is not a complete enclosure or thermal specification. For unknown optional parts it is only a placeholder. All figures exclude mating plugs and wire bends.

| Component | Nominal reference / estimate | Plausible board height | Reserve to Z | Confidence and basis |
|---|---:|---:|---:|---|
| Audio J15 | 6 | 5-8 | 9 | Low: narrow right-angle 3.5 mm plastic jack in photo; exact variant unknown. [A] is a family comparison, not footprint match. |
| VGA J12/J17 | 12.6 | 11.5-14 | 15 | Medium: HD15 female right-angle connector and mounting ears; [B] gives 12.60 body height. |
| HDMI J16 | 6.2 | 5.5-8.5 | 10 | Medium: horizontal Type A receptacle. [C] lists several standard bodies around 6.1-6.23 and a shielded 8.1 variant. |
| RJ45 J13/J14 | 12.9 | 12-14.5 | 16 | Medium: unshielded horizontal 8P8C socket; [D] gives a 12.85 height reference. |
| Dual USB J11 | 15.6 | 15-17 | 18 | Medium: shielded stacked Type A, two ports; [E] explicitly gives 15.60 above board. |
| DC barrel input JT1 | 11 | 10-12 | 13 | Medium: cylindrical right-angle jack with rectangular base; [F] gives 11.0 height, 9.0 width, 14.4 depth. Bore diameter is not identified. |
| SATA data CN3 | 8.4 | 8-10 | 11 | Medium-low: upright seven-contact SATA signal header, [G] gives 8.4 above board for through-hole type. |
| HDD power J5 | 8.5 | 8-11 | 12 | Low: four-position white power header. VH-like shape; [H] top-entry header is 8.5 high. Series/pitch unconfirmed. |
| Fan CN2 | unknown | unknown | 10 placeholder if fitted | Unpopulated in repository photo. The diagram footprint does not identify the installed part. |
| Front-panel J3/J4 region | 6 | 5-9 | 10 | Low: small white multipole shrouded header; PH/XH comparison only [I,J]. No exact pitch match established. |
| USB extension CN4 | 7 | 6-9 | 10 | Low: white four-position top-entry shrouded header; XH-like comparison [I]. |
| Speaker J8 | unknown | unknown | 10 placeholder if fitted | Pads appear unpopulated in repository photo; a fitted header/cable could differ. |
| 433 module J2 | 7 | 6-9 | 10 | Low: white five-position shrouded header; PH/XH family comparison [I,J], pitch unconfirmed. |
| BT1 holder | 24.9 reported | tolerance unspecified | 27 preliminary | Owner measurement; retains previously reported dimensions. |
| Heatsink | 22.4 reported | tolerance/datum unspecified | 25 only if PCB-to-top | Owner measurement. If 22.4 is heatsink body height alone, add the chip/adhesive/stand-off elevation before using it as installed Z. |

The white-header comparisons deliberately distinguish bare headers from mated assemblies: JST XH bare top-entry headers are about 7 mm high, while the mated system is about 9.8 mm; PH bare top-entry headers are about 6 mm, while the mated system is about 8 mm. A wire leaving vertically needs additional bend space. A SATA cable and its strain relief can exceed the bare 11 mm reservation substantially.

## Other visible parts and enclosure implications

- The photograph also shows low-profile ICs, a tilted metal-can crystal, inductors, and an electrolytic capacitor near the power jack. Their exact packages and heights have not been identified reliably. No individual numerical heights are claimed for them.
- The front-panel and RF headers in the photograph are identified by family only. Do not order mating plugs using these comparison part names without measuring pitch and keying.
- The measured BT1 envelope currently controls the reported height at 24.9 mm. A ceiling at Z=27 mm would leave 2.1 mm over that holder **if the datum is correct**, but this is not proof that the whole populated board, mated cables or heatsink assembly fits. Cooling and ventilation need a separate allowance.
- Overall enclosure height also includes support height below the board, PCB thickness, underside solder/pin clearance, and enclosure wall thickness. These remain unmeasured.
- Retain the X/Y image estimates already recorded; a reference manufacturer's different footprint should not silently replace the board-derived geometry.

[Machine-readable height estimates](component-height-estimates.csv)

## Comparison drawings and specifications

A. [Same Sky SJ1-351X audio jack, mechanical drawing page 2](https://www.sameskydevices.com/product/resource/sj1-351x.pdf). This wider threaded jack has a 6 mm body height; the photographed narrow plastic jack is not this exact part. Used only to support a broad family-level height estimate.

B. [NorComp 181 female right-angle HD D-sub drawing, page 1, dimension F=12.60](https://content.norcomp.net/rohspdfs/Connectors/18Y/181/181-YYY-213RYY1.pdf). Width 30.80 mm is reasonably close to the diagram's approximately 31.2 mm VGA envelope.

C. [Amphenol HDMI/DisplayPort product presentation, HDMI 2.1 Type A product table](https://cdn.amphenol-cs.com/media/wysiwyg/files/documentation/customerpresentation/hdmi-displayport-productpresentation.pdf). Used as geometry comparisons; HDMI version and installed part are not identified by appearance.

D. [Wurth WR-MJ 615008148521 horizontal plastic 8P8C jack, drawing page 1](https://www.we-online.com/components/products/datasheet/615008148521.pdf).

E. [Amphenol 72309-3045RPLF stacked USB 2.0 Type A, height above board 15.60 mm](https://www.amphenol-cs.com/product/723093045rplf.html).

F. [CUI PJ-002A manufacturer drawing hosted by Mouser, page 2](https://www.mouser.com/datasheet/2/670/pj_002a-1778764.pdf). Used for geometry only; its 2.0 mm center pin does not establish the board jack's mating size.

G. [Delphi commercial connector catalog, Serial ATA board connector section](https://www.mouser.com/catalog/supplier/library/pdf/Delphiconnectorscatalog.pdf). Manufacturer catalog specifies 8.4 mm above board for vertical through-hole SATA signal connectors.

H. [JST VH drawing, page 3](https://www.jst-mfg.com/product/pdf/eng/eVH.pdf): 8.5 mm bare top-entry header height. A similar outline does not prove VH compatibility.

I. [JST XH header drawings](https://order.jst-mfg.com/InternetShop/app/pdf_show.php?kbn=1&key=eXH.pdf): bare header versus mated-system dimensions.

J. [JST PH drawing](https://www.jst-mfg.com/product/pdf/eng/ePH.pdf): bare header versus mated-system dimensions.
