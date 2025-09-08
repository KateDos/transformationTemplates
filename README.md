# Microsoft FHIR Converter with PIQI Custom Templates

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Creating Custom Liquid Templates](#creating-custom-liquid-templates)
- [Running the Converter](#running-the-converter)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

---

## Overview
The **Microsoft FHIR (Fast Healthcare Interoperability Resources) Converter** is an open-source tool designed to convert healthcare data between various formats.  
This project extends it with **custom Liquid templates** that convert FHIR R4 JSON resources into **PIQI format**.

---

## Prerequisites
Before you begin, ensure you have:

- [.NET 8.0 SDK](https://dotnet.microsoft.com/download) installed  
- A basic understanding of **FHIR standards** and **JSON data formats**  
- Familiarity with **Liquid templating syntax**  
- Knowledge of the **PIQI format** (see [PIQI Framework](https://piqiframework.org/))  

---

## Getting Started

1. **Clone the Repository**
   ```bash
   git clone https://github.com/microsoft/fhir-converter.git
   cd fhir-converter
    ```

2. **Restore Dependencies**
   Restore the project dependencies by running:
   ```bash
   dotnet restore
   ```

## Project Structure

The Microsoft FHIR Converter project has a specific structure to support custom Liquid templates:
```bash
    fhir-converter/
    ├── src/                     # Converter source code
    └── data/
        └── Templates/           # Default and custom Liquid templates
```

## Creating Custom Liquid Templates

1. **Template File Structure**

   Each template folder should contain:
    - A Json/ folder with:
        - metadata.json
        - Schema definitions
    - Conversion format files with Liquid mapping rules that describe how FHIR elements map to PIQI elements
  

2. **Creating a New Template Directory**
   Copy the provided `FHIRToPIQI` folder to `data/Templates/`.
    ```
     data/Templates/
    ├── Json/
    │   └── metadata.json
    ├── FHIRToPIQI/
    ├── _patient.liquid
    ├── _observation.liquid
    └── ...
    ```


3. **Define the Schema and Liquid Mapping Rules**

    For example, `data/Templates/FHIRToPIQI/_patient.liquid` maps FHIR R4 Patient resources to PIQI demographics:
   ```liquid
   {
     "id": "{{ id }}",
     "name": "{{ name[0].given[0] }} {{ name[0].family }}",
     "gender": "{{ gender }}",
     "birthDate": "{{ birthDate }}"
   }
   ```

## Running the Converter

Use the FhirProcessor.Convert method to run conversions with custom templates.

```c#
private async Task<string> TransformFile(string data)
{
	string rootTemplate = "BundleJSON";
			
	ILogger<FhirProcessor> logger = _loggerFactory.CreateLogger<FhirProcessor>();
	string transformDirectory = Path.Combine(AppContext.BaseDirectory, @"Transforms\FHIRToPIQI");
	Microsoft.Health.Fhir.Liquid.Converter.Models.DataType dataType = Util.GetDataTypes(transformDirectory);
	Microsoft.Health.Fhir.Liquid.Converter.Processors.FhirProcessor dataProcessor = new FhirProcessor(new ProcessorSettings(), logger);
	TemplateProvider templateProvider = new TemplateProvider(transformDirectory, dataType);

	return dataProcessor.Convert(data, rootTemplate, templateProvider, null);       
}
    
private static DataType GetDataTypes(string templateDirectory)
{
    if (!Directory.Exists(templateDirectory))
    {
        throw new DirectoryNotFoundException($"Could not find template directory: {templateDirectory}");
    }

    var metadataPath = Path.Join(templateDirectory, MetadataFileName);
    if (!File.Exists(metadataPath))
    {
        throw new FileNotFoundException($"Could not find metadata.json in template directory: {templateDirectory}.");
    }

    var content = File.ReadAllText(metadataPath);
    var metadata = JsonConvert.DeserializeObject<Metadata>(content);
    if (Enum.TryParse<DataType>(metadata?.Type, ignoreCase: true, out var type))
    {
        return type;
    }

    throw new NotImplementedException($"The conversion from data type '{metadata?.Type}' to FHIR is not supported");
}
```
**Convert Method Signature**
```c#
public string Convert(
    string data,
    string rootTemplate,
    ITemplateProvider templateProvider,
    CancellationToken cancellationToken,
    TraceInfo traceInfo = null
)
```



## Troubleshooting

- **Template Not Found**: Ensure your template folder and files are correctly named and placed in the `data/Templates/` directory.
- **Liquid Syntax Issues**: Validate your Liquid templates and check mappings against FHIR JSON structure.

## Additional Resources

- [Microsoft FHIR Converter GitHub](https://github.com/microsoft/fhir-converter)
- [FHIR Specification](https://www.hl7.org/fhir/)
- [Liquid Template Language Documentation](https://shopify.dev/themes/liquid)
- [Azure Logic Apps - Liquid transforms](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-liquid-transform?tabs=consumption) This would be an alternate way to use the templates to convert to PIQI format.

If you encounter any issues, please open an issue on the [PIQI Alliance Github](https://github.com/piqiframework) 

---

*Last Updated: 09/05/2025*
