Cooling
#######

.. contents:: Table of Contents

.. _primaryclgsys:

Determining the primary cooling system
**************************************

HEScore only allows the definition of one cooling system. If an HPXML document
contains more than one cooling system then the primary one must be chosen for
input into HEScore. The primary cooling system is determined according to the
following logic:

#. HPXML has a ``PrimaryCoolingSystem`` element that references with system
   is the primary one. If this is present, the properties of that referenced
   cooling system are translated into HEScore inputs.
#. If there is no defined primary heating system in HPXML, the
   ``CoolingSystem`` or ``HeatPump`` with the greatest cooling capacity is
   used. 
#. If neither of the above conditions are met the first ``CoolingSystem`` or
   ``HeatPump`` in the document is used.
#. Finally, if there is no ``CoolingSystem`` or ``HeatPump`` object, then the
   house is determined to not have a cooling system in HEScore. 

.. warning::

   The translation is not currently doing the weighted average of like systems 
   as described in the HEScore help.
   
Heating system type
*******************

HPXML provides two difference HVAC system elements that can provide cooling:
``CoolingSystem`` that only provides cooling and ``HeatPump`` which can provide
heating and cooling. 

Heat Pump
=========

The ``HeatPump`` element in HPXML can represent either an air-source heat pump
or ground source heat pump in HEScore. Which is specified in HEScore is
determined by the ``HeatPumpType`` element in HPXML according to the following
mapping.

.. table:: Heat Pump Type mapping

   ============================  ============================
   HPXML Heat Pump Type          HEScore Heating Type
   ============================  ============================
   water-to-air                  heat_pump
   water-to-water                heat_pump
   air-to-air                    heat_pump
   mini-split                    heat_pump
   ground-to-air                 gchp
   ============================  ============================

Cooling System
==============

The ``CoolingSystem`` element in HPXML is used to describe any system that
provides cooling that is not a heat pump. The ``CoolingSystemType`` subelement
is used to determine what kind of cooling system to specify for HEScore. This
is done according to the following mapping.

.. table:: Cooling System Type mapping

   =========================  ====================
   HPXML Heating System Type  HEScore Heating Type
   =========================  ====================
   central air conditioning   split_dx
   room air conditioner       packaged_dx
   mini-split                 split_dx
   evaporative cooler         dec
   =========================  ====================

.. note::
   
   If "other" is selected as the cooling system type in HPXML, the 
   translation will error out.

.. warning::

   There is no way to specify an indirect or direct/indirect evaporative cooler 
   in HPXML, so those choices in HEScore are not available 
   through this translation. As shown above, all evaporative coolers in 
   HPXML are assumed to be direct.

Cooling Efficiency
******************

Cooling efficiency can be described in HEScore by either the rated efficiency
(SEER, EER), or if that is unavailable, the year installed/manufactured from
which HEScore estimates the efficiency based on shipment weighted efficiencies
by year. The translator follows this methodology and looks for the rated
efficiency first and if it cannot be found sends the year installed. 

Rated Efficiency
================

HEScore expects efficiency to be described in different units depending on the
cooling system type. 

.. table:: HEScore cooling type efficiency units

   ===============  ================
   Cooling Type     Efficiency Units
   ===============  ================
   split_dx         SEER
   packaged_dx      EER
   heat_pump        SEER
   gchp             SEER
   dec              *not translated*
   iec              *not translated*
   idec             *not translated*
   ===============  ================

.. warning::

   It is unclear from the :term:`API` documentation as well as the HEScore
   user interface how to specify efficiency for evaporative coolers. Efficiency
   is omitted from the cooling system specification in HEScore if it is 

The translator searches the ``CoolingSystem/AnnualCoolingEfficiency`` or
``HeatPump/AnnualCoolEfficiency`` elements of the primary cooling system and
uses the first one that has the correct units.

Shipment Weighted Efficiency
============================

When an appropriate rated efficiency cannot be found, HEScore can accept the
year the equipment was installed and estimate the efficiency based on that. The
year is retrieved from the ``YearInstalled`` element, and if that is not
present the ``ModelYear`` element. 

