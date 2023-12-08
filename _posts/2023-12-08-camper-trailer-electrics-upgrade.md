---
permalink: /camper-trailer-electrics-upgrade
title: "Electrifying Adventures: Upgrading our 2017 Stoney Creek Camper"
description: "A blog post about upgrading the electrics in our 2017 Stoney Creek SC-FF6 camper trailer"
image:
    path: "/assets/img/battery-leads.jpg"
    alt: "Battery Cables"
categories: [Camper Trailer]
tags: [[camper trailer, victron, battery, lifepo4]]
last_modified_at: 2023-12-08
---

## The Disclaimer

> Please be advised that electrical work, particularly in recreational vehicles such as camper trailers, involves significant risks and should only be undertaken by qualified individuals. It is strongly recommended that any electrical upgrades, installations, or repairs be performed under the supervision of a certified electrician or a professional with relevant experience.
>
> Electrical systems in camper trailers are complex and can pose serious hazards if not handled properly, including risks of electric shock, fire, damage to equipment, and voiding of warranties. It is crucial to adhere to all safety standards and regulations.
>
> Additionally, always consult the manufacturer’s guidelines and local regulations before making any modifications to your camper’s electrical system. Improper modifications can lead to dangerous situations and may also have legal implications.
>
> This blog shares our personal experiences and is intended for informational purposes only. It should not be taken as professional advice. We are not responsible for any damage, injury, or legal issues that arise from using the information provided in our posts. Always seek professional assistance for electrical work to ensure safety and compliance.
{: .prompt-danger }

## The Beginning of a Journey

Back in 2022, our family's adventurous spirit led us to acquire a second-hand 2017 Stoney Creek SC-FF6 camper trailer. While the trailer had undergone some modifications in its relatively short life, it still sported an essentially original build. The heart of its power system was a basic 12V electrical setup, equipped with two 100Ah AGM batteries and a CTEK M200 AC charger. This system was intended to support a 75L Evakool camp fridge, interior and exterior LED lighting, and two cold water pumps, each connected to its own tank. A somewhat risky addition was a 50A Anderson plug at the drawbar, connected directly to the batteries without any fuse – presumably an aftermarket upgrade by the previous owner for a Redarc DCDC charger linked to their tow vehicle.

![Our 2017 Stoney Creek SC-FF6 camper trailer](/uploads/2023-12-08-camper-trailer-electrics-upgrade/camper.jpg){: width="500" }
_Our 2017 Stoney Creek SC-FF6 camper trailer_

## Identifying the Need for an Upgrade

As we began to use the trailer more frequently, it became apparent that the existing electrical system had its limitations. The AGM batteries, due to their age, lacked the capacity to meet our energy demands, especially during extended trips. Since our tow vehicle does not have an Anderson plug to charge the camper on the road, on the longer trips the fridge would cut out from low voltage. While caravan parks we were staying in provided the opportunity to recharge from AC supply, we were not be able to go off grid for very long at all.

## Choosing the Right Components

After extensive research (Google, lol), we decided to upgrade to a more robust and efficient system. Our choice fell on LiFePO4 batteries, known for their longer lifespan, better performance, and enhanced safety compared to traditional AGM batteries. For the heart of our new system, we chose Victron equipment, renowned for its reliability and advanced features.

## A Little Bit of Planning

In my professional life, I delve deep into the world of computer networking, where a well-organized, logical, and clear network schematic is not just a tool, but a source of genuine satisfaction. The value of proper documentation in designing, installing, and commissioning network systems cannot be overstated—it simplifies processes, prevents missteps, and fosters efficiency (quite an obvious yet often overlooked aspect, right?). Drawing from this professional ethos, I approached our camper trailer electrical upgrade with the same level of meticulousness. So, without further ado, let me present to you the carefully crafted block diagram of our electrical system upgrade!

![Electrical upgrade schematics](/uploads/2023-12-08-camper-trailer-electrics-upgrade/schematic.jpg){: width="500" }
_Electrical upgrade schematic_

The upgrade encompasses a comprehensive set of enhancements, including:

 - Replacing original AGM shunt with Victron BMV-712 and SmartShunt 500A.
 - Replacing worn out 100Ah AGMs with new VoltX 100Ah Premium Plus LiFePO4 batteries.
 - Upgrade high current cabling to '0 B&S' size.
 - Replacing CTEK M200 AC charger with Victron Blue Smart IP22 12 \| 30 AC charger.
 - New 250A Class T fuse and mount.
 - New negative and postitive busbars to help dress wiring and reduce the number of 'stacked' ring terminals.
 - New Victron SmartSolar MPPT 100 \| 30 charge controller.
 - New 40A circuit breaker between batteries and solar charger.

 These upgrades collectively aim to not only enhance the efficiency and safety of the camper trailer’s electrical system but also to prepare it for more demanding off-grid adventures.

![Old CTEK M200 AC charger removed](/uploads/2023-12-08-camper-trailer-electrics-upgrade/old-charger.jpg){: width="250" }
_Old CTEK M200 AC charger removed_

 ![VoltX 100Ah LiFePO4 Premium Plus battery](/uploads/2023-12-08-camper-trailer-electrics-upgrade/battery.jpg){: width="250" }
_VoltX 100Ah LiFePO4 Premium Plus battery_

 ![Distribution blocks](/uploads/2023-12-08-camper-trailer-electrics-upgrade/distribution-blocks.jpg){: width="250" }
_Distribution blocks_

 ![Fuse arrangement](/uploads/2023-12-08-camper-trailer-electrics-upgrade/big-fuse.jpg){: width="250" }
_Fuse arrangement_

 ![Victron SmartSolar MPPT 100 \| 30 charge controller](/uploads/2023-12-08-camper-trailer-electrics-upgrade/mppt-charger.jpg){: width="250" }
_Victron SmartSolar MPPT 100 \| 30 charge controller_

 ![Blue Smart IP22 12 \| 30 AC charger](/uploads/2023-12-08-camper-trailer-electrics-upgrade/new-charger.jpg){: width="250" }
_Victron Blue Smart IP22 12 \| 30 AC charger_

## The Results

The results of the upgrade were immediately noticeable. The LiFePO4 batteries provided a significant increase in usable capacity, allowing us to power our appliances for longer periods without the need for frequent recharging. The Victron equipment brought a new level of sophistication to our setup, with the ability to monitor and control various aspects of the system through a smartphone app.

To thoroughly test the effectiveness of our new setup, we conducted a runtime experiment exclusively using the 75L Evakool fridge. Previously, with the old batteries, the fridge's runtime was limited to approximately 6 hours. However, with the upgraded batteries, the fridge sustained over 5 days of continuous operation before I chose to conclude the test. This significant improvement in performance is quite remarkable, in my opinion and speaks to the advantages of LiFePO4 over AGM battery technologies.

![Runtime test](/uploads/2023-12-08-camper-trailer-electrics-upgrade/runtime-test-1.jpg){: width="250" }
_Runtime test_

![Runtime test](/uploads/2023-12-08-camper-trailer-electrics-upgrade/runtime-test-2.jpg){: width="250" }
_Runtime test_

Our upgraded system now enables us to confidently venture off-grid, powered by a 160W folding solar panel. During a recent excursion to central Victoria, this solar charging arrangement exceeded our expectations. Impressively, it achieved a full charge by midday - a rapid recovery from the previous night's energy consumption. This efficiency not only highlights the effectiveness of our new setup but also underscores our capability for sustained off-grid adventures, free from the constraints of traditional power sources.

![Solar charging](/uploads/2023-12-08-camper-trailer-electrics-upgrade/solar.jpg){: width="250" }
_Solar charging_

## Future Plans

As illustrated in the schematic, we have aspirations to enhance the system further by integrating a Victron Phoenix Inverter Smart 12 \| 1600. However, up to this point, our off-grid adventures haven't necessitated the use of 230V AC power. Although this upgrade remains on our wish list, its implementation is currently on hold until such a need arises.

## A Game-Changer for Our Camping Experience

This upgrade has been a game-changer for our camping experience. The enhanced power capacity and efficiency have opened up new possibilities for off-grid adventures, and the peace of mind provided by the advanced safety features is invaluable. We look forward to many more journeys with our newly empowered Stoney Creek camper, exploring the great outdoors with confidence and comfort.
