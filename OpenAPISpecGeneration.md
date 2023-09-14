# OpenAPI Spec Generation Proposal

This is a document outlining a proposal for the generation of an OpenAPI spec from an existing Laravel API (The company's API).

The author of this document would choose Solution 1 but has presented the other options for completeness.

## Purpose and Benefits

An OpenAPI spec provides a contract for the API to fulfil and allows consumers of the API to utilise it in a consistent, transparent manner.

An OpenAPI spec can be used to automate the generation of acceptance tests for the API, enabling CI/CD and validating that the system maintains its contracts.

Human-readable API documentation can be generated from the API spec (through Swagger or others), providing internal developers and potential consumers a fully browsable catalogue of the API's functionality.

Variations and additions to the spec file can effectively be used as requirements documentation for new features, with auto-generated tests facilitating TDD.

## The Problem Space

The spec is to be generated for a Laravel API with some legacy documentation and a low level of test coverage. It is unknown whether the company's API has Model Factories or Seeders configured for its current database. It is assumed that these are either not in place or incomplete. The legacy documentation is not in a format easily convertible to OpenAPI. 

An OpenAPI spec is to be generated from this API, ideally in a fully automated fashion.

## Solutions in Summary

[1.](#generate-from-seeded-database) Generate the documentation by utilising Laravel's seeders and factories to create a comprehensive database state, i.e. a state in which realistic variations of each individual model and its relations are represented. Model specs can be introspected from this database and routes can be probed to assess the structure of responses the API will provide to standard REST operations.

[2.](#generate-with-docs-converter) Generate the documentation by writing a converter for the legacy docs to the OpenAPI spec. As the legacy docs are out of step with the latest API, additional functionality or variations to existing functionality must be detailed through manual editing.

[3.](#generate-from-har-file) Use a tool (browser extension or Postman etc.) to generate a HAR file capturing a comprehensive set of requests to the API and its responses to these. These requests are to be generated in-app through navigation and form-filling. A spec file can then be generated from this HAR file (a variety of tools exist to do this generation).

[4.](#generate-using-attributes-and-package) This [package](https://github.com/vyuldashev/laravel-openapi) provides for the automatic generation of OpenAPI documentation once Controller methods and classes are appropriately decorated with attributes.

[5.](#create-manually) Write the spec manually based on the legacy docs, the current database structure and the conventions followed in a representative subset of the controllers.

## Solutions in Detail

1. ### Generate from Seeded Database

**Steps**
1. Write a comprehensive set of model factories in Laravel.
1. Write a set of seeders that produce a comprehensive database state.
1. Seed the database.
1. Code a service that accepts a list of Models as classnames and introspects the datatypes of each field in the model from the database through Laravel's built-in schema builder. This will generate object schemas for use in the OpenAPI Spec.
1. Code a service that generates request bodies based on the object schemas generated above.
1. Code a service that generates a standard set of error responses for 403, 401, 405, 404 etc as response schemas.
1. Code a service that accepts a list of routes files or a directory where routes files are contained and then runs all operations specified in each route file. Request bodies generated above should be used to probe the creative and mutative endpoints. Both object stubs and invalid objects should be included in the requests to test the API's response to varied input. For the schema of invalid responses, assumptions will be made based on domain knowledge, e.g. if the standard invalid response is a 422 with an object containing invalid fields as properties and arrays of error messages as values, the response schema will likely be identical to the requestBody schema for the endpoint.
1. Code a service that creates a file per resource type (Model), which file contains all component schemas, request bodies, and response schemas for that resource type. This service also creates a main openapi file, with a paths section which references the individual files per resource type using $ref notation. All the information used to generate these files is passed from the service described above and the information is written using standard PHP yaml functions.
1. Use the [OpenAPI HttpFoundation testing package](https://github.com/osteel/openapi-httpfoundation-testing) to produce a suite of tests for the API based on the generated spec file. This will root out any inconsistencies.
1. Manually edit anything the spec generator gets wrong.

**Considerations**
- Model factories and seeders are also useful for CI/CD as they make it easier to generate test cases and provide a base against which to continually validate the API against its spec.
- Similar to the above, some of the work undertaken here is also useful in the paying of technical debt for the system as it can be used in the generation of unit, functional and integration tests.
- For maintenance of this spec, future feature requests / issues should have as their first step the alteration of the spec file. Tests can be automatically generated to validate the new spec and thus the spec can lead the development in a similar way to TDD.
- This is likely to be a complex endeavour and once completed, most of the code won't be used again. It is possible that the code could be reused for generating OpenAPI specs for any prototype projects the company is working on, or it could be open-sourced for the PHP community if made more general.
- This solution is a little bit uncertain in terms of how much functionality can actually be inferred from the database and probing of the endpoints. Some non-obvious states and responses are likely to be missed.

2. ### Generate with Docs Converter

**Steps**
1. Assess the legacy docs' conformance with reality and make good any defects.
1. (Unknown if required) Write a parser for the legacy docs format.
1. Write a set of general converter classes for entities in the legacy docs with a convert method for converting these into entities in the OpenAPI spec.
1. Write a generator to turn the converted entities into OpenAPI specs.
1. Write a generator for any information not contained within the legacy docs, e.g. paths, actions, standard response types.
1. Write a service to generate a main OpenAPI file and sub-files representing either the resource type or the grouping of functionality in the legacy docs.
1. Edit the files manually if there are manifest incompatibilities between the two formats.

**Considerations**
- Making good the legacy docs provides a valid source of truth for API consumers, even if it is for an interim period.
- Legacy docs might be too incompatible with the OpenAPI structure for this to be a worthwhile endeavour.

3. ### Generate from HAR File

**Steps**
1. A developer or QA operative with significant domain knowledge navigates the site in a comprehensive fashion, executing valid and invalid requests and capturing these in a HAR file, or a developer uses the routes file to generate a collection of Postman requests that can be run against the API to generate a HAR file or even the OpenAPI spec itself.
1. [OpenAPI Tools](https://openapi.tools/) contains multiple tools that can be used to generate an OpenAPI spec from HAR files. Evaluate these and then generate the spec.
1. A developer with significant domain knowledge assesses the generated file for completeness, as that is the most likely defect.
1. Repeat the above process with a more complete set of requests or manually add the missing functionality.

**Considerations**
- Some API functionality is difficult to model in a front-end and may require significant additional setup for the database to reach a state in which a test can be done.
- The first step can potentially be outsourced to QA, which may be more cost-effective.

4. ### Generate Using Attributes and Package

**Steps**
1. Learn how to use the [package](https://github.com/vyuldashev/laravel-openapi)
1. Add attributes to all Controller methods in line with the package above
1. Generate the OpenAPI spec from the modified code
1. Spec can then be altered either through manual editing or through the editing and addition of attributes from the package above to PHP controllers

**Considerations**
- This is a PHP only solution, so should be doable by most PHP developers.
- Domain knowledge would be beneficial to the speed of completion. The responses and request bodies may not be obvious from the code.
- This requires the learning of a Domain Specific Language in addition or instead of the OpenAPI spec itself.
- This creates a hard dependency on the package above as it would be relatively painful to remove all the attributes.

5. ### Create Manually

**Steps**
1. Draw up component schemas for all Models in the framework.
1. If request bodies use stub schemas, create these.
1. Write schemas for all common response bodies: 401, 403, 404, 405 etc.
1. For each Model, create a file containing all component schemas, stub schemas and response bodies related to that model.
1. In the main OpenAPI file paths section, reference the schemas in each of these files for the requestBodies, parameters and responses.


**Considerations**
- A simple spec generator could be coded to concatenate the paths for each Model with the main OpenAPI file. This would help when dealing with a very large main file.
- This strategy is likely to be efficient if there are a low number of Models.
- The generated spec file doesn't have to be finished all at once, which the other solutions more or less require. A valid file that describes only a subset of the API's functionality is still useful.
- Once fully generated, the files should be relatively easy to maintain.


