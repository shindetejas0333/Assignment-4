# Assignment-4
# Input values
fck = float(input("Enter the value of characteristic compressive strength (MPa): "))
Gca = float(input("Enter the value of specific gravity of CA: "))
Gfa = float(input("Enter the value of specific gravity of FA: "))
Gc = float(input("Enter the value of specific gravity of Cement: "))
Water_Density = float(input("Enter the value of Water Density (kg/m^3): "))
AGG_Size = float(input("Enter the nominal size of Aggregate (mm): "))
Nature_of_AGG = input("Nature of Aggregates (Sub-Angular/Gravel/Round): ")
Slump = float(input("Enter the value of workability of concrete (mm): "))
Admixture = input("Type of Admixture (Plastisizer/Super-plastisizer): ")
Exposure_Condition = input("Exposure Condition (Mild/Moderate/Severe/Very Severe/Extreme): ")
# Convert Exposure_Condition to title case to handle case sensitivity
Exposure_Condition = Exposure_Condition.title()
Concreting = input("Type of Concreting (Plain/Reinforced): ")
Zone = int(input("Zone (1/2/3/4): "))

# Target Mean Strength
sigma = {
    10: 3.5,
    15: 3.5,
    20: 4,
    25: 4,
    30: 5,
    35: 5,
    40: 5,
    45: 5,
    50: 5
}

if fck not in sigma:
    raise ValueError("Invalid characteristic compressive strength value.")

ft = fck + sigma[fck] * 1.65
print("Target Mean Strength: ", ft, "MPa")

# Maximum free Water Cement Ratio
WC_ratio = {
    "Plain": {
        "Mild": 0.6,
        "Moderate": 0.6,
        "Severe": 0.5,
        "Very Severe": 0.45,
        "Extreme": 0.4
    },
    "Reinforced": {
        "Mild": 0.55,
        "Moderate": 0.5,
        "Severe": 0.45,
        "Very Severe": 0.45,
        "Extreme": 0.4
    }
}

if Concreting not in WC_ratio or Exposure_Condition not in WC_ratio[Concreting]:
    raise ValueError("Invalid Concreting type or Exposure Condition.")

WC_ratio_value = WC_ratio[Concreting][Exposure_Condition]
print("W/C Ratio:", WC_ratio_value)

# Minimum Cement Content
Min_Cement_Content = {
    "Plain": {
        "Mild": 220,
        "Moderate": 240,
        "Severe": 250,
        "Very Severe": 260,
        "Extreme": 280
    },
    "Reinforced": {
        "Mild": 300,
        "Moderate": 300,
        "Severe": 320,
        "Very Severe": 340,
        "Extreme": 360
    }
}

if Concreting not in Min_Cement_Content or Exposure_Condition not in Min_Cement_Content[Concreting]:
    raise ValueError("Invalid Concreting type or Exposure Condition.")

Min_Cement_Content_value = Min_Cement_Content[Concreting][Exposure_Condition]
print("Minimum Cement Content:", Min_Cement_Content_value, "kg/m^3")

# Water Content
Water_Content = {
    10: 208,
    20: 186,
    40: 165
}

if AGG_Size not in Water_Content:
    raise ValueError("Invalid Aggregate Size.")

Water_Content_value = Water_Content[AGG_Size]

if Slump == 75:
    Water_Content_value += Water_Content_value * 0.03
elif Slump == 100:
    Water_Content_value += Water_Content_value * 0.06
elif Slump == 125:
    Water_Content_value += Water_Content_value * 0.09
elif Slump == 150:
    Water_Content_value += Water_Content_value * 0.12
elif Slump == 175:
    Water_Content_value += Water_Content_value * 0.15
elif Slump == 200:
    Water_Content_value += Water_Content_value * 0.18

if Nature_of_AGG == "Sub-Angular":
    Water_Content_value -= 10
elif Nature_of_AGG == "Gravel":
    Water_Content_value -= 20
elif Nature_of_AGG == "Round":
    Water_Content_value -= 25

if Admixture == "Plastisizer":
    Water_Content_value -= (0.1 * Water_Content_value)
elif Admixture == "Super-plastisizer":
    Water_Content_value -= (0.2 * Water_Content_value)

print("Water Content:", Water_Content_value, "kg/m^3")

# Cement Content
Cement_Content = Water_Content_value / WC_ratio_value
print("Cement Content:", Cement_Content, "kg/m^3")
print("As per IS 456:2000, Maximum allowed Cement Content is 450 kg/m^3")

if Cement_Content > 450:
    Cement_Content = 450
print("Cement Content (adjusted):", Cement_Content, "kg/m^3")

# Volume Calculations
Vol_Cement = Cement_Content / (Gc * Water_Density)
print("Volume of Cement:", Vol_Cement, "m^3")

Vol_Water = Water_Content_value / Water_Density
print("Volume of Water:", Vol_Water, "m^3")

Vol_AGG = 1 - Vol_Water - Vol_Cement
print("Volume of Coarse Aggregates and Fine Aggregates:", Vol_AGG, "m^3")

# Zone ID
Zone_ID = {
    1: {10: 0.44, 20: 0.60, 40: 0.69},
    2: {10: 0.46, 20: 0.62, 40: 0.71},
    3: {10: 0.48, 20: 0.64, 40: 0.73},
    4: {10: 0.50, 20: 0.66, 40: 0.75}
}

if Zone not in Zone_ID or AGG_Size not in Zone_ID[Zone]:
    raise ValueError("Invalid Zone or Aggregate Size.")

Fraction = Zone_ID[Zone][AGG_Size]

if WC_ratio_value == 0.5:
    Fraction = Fraction
elif WC_ratio_value == 0.45:
    Fraction += 0.01 * Fraction
elif WC_ratio_value == 0.4:
    Fraction += 0.02 * Fraction
elif WC_ratio_value == 0.55:
    Fraction -= 0.01 * Fraction
elif WC_ratio_value == 0.60:
    Fraction -= 0.02 * Fraction

print("Coarse Aggregate Fraction:", Fraction)

Vol_CA = Vol_AGG * Fraction
print("Volume of Coarse Aggregate:", Vol_CA, "m^3")

Vol_FA = Vol_AGG - Vol_CA
print("Volume of Fine Aggregate:", Vol_FA, "m^3")

Mass_CA = Vol_CA * Gca * Water_Density
print("Mass of Coarse Aggregates:", Mass_CA, "kg")

Mass_FA = Vol_FA * Gfa * Water_Density
print("Mass of Fine Aggregates:", Mass_FA, "kg")

# Ratios
print("Weight Batching")
print(f"{Cement_Content/Cement_Content:.2f} : {Mass_FA/Cement_Content:.2f} : {Mass_CA/Cement_Content:.2f} : {Water_Content_value/Cement_Content:.2f}") # Use Water_Content_value (float) instead of Water_Content (dict)

print("Volume Batching:")
print(f"{Vol_Cement/Vol_Cement:.2f} : {Vol_FA/Vol_Cement:.2f} : {Vol_CA/Vol_Cement:.2f} : {Vol_Water/Vol_Cement:.2f}")
