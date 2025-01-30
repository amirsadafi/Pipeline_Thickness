# Pipeline_Thickness

Pipeline Sizing: Engineering Procedure

This Python script implements a pipeline sizing algorithm based on an engineering flowchart. 
It uses fundamental equations, including the Continuity Equation, Reynolds Number, and 
Darcy-Weisbach equations, to calculate pipe diameter, velocity, Reynolds number, friction factor, 
and pressure drop. The script iteratively adjusts the pipe diameter to ensure compliance with 
maximum allowable velocity and pressure drop constraints.

Additionally, this script includes pipe wall thickness calculations using:
- **Barlow’s Formula** (for thin-walled pipes)
- **ASME B31.3 Formula** (for process piping)

Features:
- Calculates optimal pipe diameter for a given mass flow rate and fluid properties.
- Computes velocity, Reynolds number, and friction factor.
- Ensures constraints on velocity and pressure drop are satisfied.
- Calculates required pipe thickness based on pressure, material strength, and design standards.

Usage:
Provide inputs such as:
- Mass flow rate (kg/s)
- Fluid density (kg/m^3)
- Fluid viscosity (Pa.s)
- Pipe length (m)
- Maximum allowable pressure drop (Pa)
- Maximum allowable velocity (m/s)
- Internal pressure (Pa)
- Pipe outer diameter (m)
- Allowable stress (Pa)
- Weld efficiency factor (unitless)
- Corrosion allowance (m)
- Material roughness (m)
- Pipe internal diameter (m)
- Gravity acceleration (m/s²)
- Pipe absolute roughness (m)

The functions return the computed parameters for the pipeline.

Example:
Call the `pipeline_sizing` function with your specific parameters and print the results.
Call `pipe_thickness_calculation` function to determine pipe thickness requirements.

Friction Factor Calculation:
- For laminar flow (Re < 2000), the friction factor is calculated using \( f = 64/Re \).
- For turbulent flow, an approximation using the Blasius equation is used.
- For a more precise turbulent flow calculation, refer to the **Moody Diagram** or the **Colebrook Equation**.

This script is ideal for engineers and designers working on pipeline systems in various industries.

"""
import math

def pipeline_sizing(mass_flow_rate, density, viscosity, pipe_length, max_pressure_drop, max_velocity):
    """
    Perform pipeline sizing based on engineering procedures.

    Parameters:
        mass_flow_rate (float): Mass flow rate (kg/s).
        density (float): Fluid density (kg/m^3).
        viscosity (float): Fluid dynamic viscosity (Pa.s).
        pipe_length (float): Length of the pipeline (m).
        max_pressure_drop (float): Maximum allowable pressure drop (Pa).
        max_velocity (float): Maximum allowable velocity (m/s).

    Returns:
        dict: Contains the calculated parameters (pipe diameter, velocity, Reynolds number, friction factor, pressure drop).
    """

    def reynolds_number(diameter, velocity):
        return (density * velocity * diameter) / viscosity

    def darcy_weisbach_friction_factor(re):
        if re < 2000:
            # Laminar flow
            return 64 / re
        else:
            # Turbulent flow (approximation using Blasius equation)
            return 0.3164 / (re ** 0.25)

    def pressure_drop(friction_factor, velocity, diameter):
        return friction_factor * (pipe_length / diameter) * (density * velocity**2 / 2)

    # Start sizing
    diameter = 0.05  # Initial guess for diameter in meters
    velocity = 0
    re = 0
    friction_factor = 0
    dp = 0

    while True:
        # Calculate velocity from the continuity equation
        area = math.pi * (diameter / 2) ** 2
        velocity = mass_flow_rate / (density * area)

        if velocity > max_velocity:
            # Increase diameter if velocity exceeds the maximum
            diameter += 0.01
            continue

        # Calculate Reynolds number
        re = reynolds_number(diameter, velocity)

        # Calculate friction factor
        friction_factor = darcy_weisbach_friction_factor(re)

        # Calculate pressure drop
        dp = pressure_drop(friction_factor, velocity, diameter)

        if dp > max_pressure_drop:
            # Increase diameter if pressure drop exceeds the maximum
            diameter += 0.01
            continue

        # Check if all conditions are satisfied
        if velocity <= max_velocity and dp <= max_pressure_drop:
            break

    return {
        "Pipe Diameter (m)": diameter,
        "Velocity (m/s)": velocity,
        "Reynolds Number": re,
        "Friction Factor": friction_factor,
        "Pressure Drop (Pa)": dp,
    }

def pipe_thickness_calculation(internal_pressure, outer_diameter, allowable_stress, weld_efficiency, corrosion_allowance, y_factor=0.4):
    """
    Calculate pipe wall thickness using Barlow’s Formula and ASME B31.3 Formula.

    Parameters:
        internal_pressure (float): Internal pressure in the pipe (Pa).
        outer_diameter (float): Outside diameter of the pipe (m).
        allowable_stress (float): Maximum allowable stress of the material (Pa).
        weld_efficiency (float): Weld joint efficiency (unitless, typically 0.85 - 1.0).
        corrosion_allowance (float): Additional thickness for corrosion (m).
        y_factor (float): Material-dependent factor (default is 0.4 for steel pipes).

    Returns:
        dict: Calculated thickness using Barlow’s and ASME B31.3 formulas.
    """
    
    # Bar
