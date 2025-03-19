Anvend tidligere web api ntier jeg har lavet


- rename all file content from laundrysystem to sps


- rename folders and files:
powershell 


param(
    [Parameter(Mandatory=$true)]
    [string]$BasePath
)

if (-not (Test-Path $BasePath)) {
    Write-Host "Error: Path '$BasePath' does not exist." -ForegroundColor Red
    exit
}

Write-Host "Renaming directories..." -ForegroundColor Cyan
# Rename directories first (subdirectories before parents)
Get-ChildItem -Path $BasePath -Recurse -Directory | 
    Sort-Object FullName -Descending | 
    ForEach-Object {
        if ($_.Name -match "LaundrySystem") {
            $newName = $_.Name -replace "LaundrySystem", "sps"
            $parentPath = $_.Parent.FullName
            $newFullPath = Join-Path -Path $parentPath -ChildPath $newName
            Write-Host "Renaming directory '$($_.FullName)' to '$newFullPath'"
            Rename-Item -Path $_.FullName -NewName $newName
        }
    }

Write-Host "`nRenaming files..." -ForegroundColor Cyan
# Now rename files in all directories
Get-ChildItem -Path $BasePath -Recurse -File | ForEach-Object {
    if ($_.Name -match "LaundrySystem") {
        $newName = $_.Name -replace "LaundrySystem", "sps"
        $parentPath = $_.DirectoryName
        $newFullPath = Join-Path -Path $parentPath -ChildPath $newName
        Write-Host "Renaming file '$($_.FullName)' to '$newFullPath'"
        Rename-Item -Path $_.FullName -NewName $newName
    }
}

Write-Host "`nRenaming complete." -ForegroundColor Green



- rename content of the files

python


import os
import sys

def replace_in_file(file_path, old_str="LaundrySystem", new_str="sps"):
    try:
        with open(file_path, "r", encoding="utf-8") as file:
            content = file.read()
    except UnicodeDecodeError:
        # Skipping non-text files
        return False

    if old_str not in content:
        return False

    new_content = content.replace(old_str, new_str)
    with open(file_path, "w", encoding="utf-8") as file:
        file.write(new_content)
    return True

def traverse_and_replace(directory):
    modified_count = 0
    for root, _, files in os.walk(directory):
        for file in files:
            file_path = os.path.join(root, file)
            if replace_in_file(file_path):
                print(f"Modified: {file_path}")
                modified_count += 1
    print(f"Total files modified: {modified_count}")

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python rename_script.py <directoryPath>")
        sys.exit(1)
    
    directory = sys.argv[1]
    if not os.path.isdir(directory):
        print(f"Error: '{directory}' is not a valid directory.")
        sys.exit(1)
    
    traverse_and_replace(directory)


	- remove the .vscode folder

Now i need to create a new identity database and remove the extended microsoft identity user

then i need to create a seperate database for sps data

	- Remove migration folder. 
	- Add connection strings in appsettings

i want to refactor and use the default user identity that is not extended






Thursday 27

Its too complex a task. 

i started to ask the agent to make a walking skeleton out of the web api, but i will lose too much design and patterns.



i start refactoring entity classes  myself and beginning with domain model layer



code first approach








-after creating the entities i am now asking the agent to create db context classes for ms identity db and the new sps database 



i expect its too complex for the agent and that i should refactor the identity database myself before asking this. But lets try :)

	- Now i am removing the old classes.

	- Now i am creating repos inheriting from baseservice that has all generic crud

i have create a script for this earlier, to save time when creating repos from scratch 



import os
# Define the models you want to create interfaces and repositories for
models = [
'Student',
'Education',
'SpsaCase',
'Period',
'Place',
'Diagnosis',
'EduCategory',
'StudentPayment',
'SupportingTeacher',
'OpkvalSupervision',
'EduStatus',
'EducationPeriodRate',
'TeacherPayment',
'SupportType',
]
# Directories to place the generated files
interfaces_dir = './Repos/Interfaces'
repos_dir = './Repos/Implementations'
# Ensure directories exist
os.makedirs(interfaces_dir, exist_ok=True)
os.makedirs(repos_dir, exist_ok=True)
# Function to create interface content for repositories
def create_interface_content(model):
    return f"""namespace sps.DAL.Repos.Interfaces
{{
    using sps.DAL.Entities;
    public interface I{model}Repo : IBaseRepo<{model}>
    {{
        // Add any additional methods specific to {model} if needed
    }}
}}
"""
# Function to create repository implementation content
def create_repo_content(model):
    return f"""namespace sps.DAL.Repos
{{
    using sps.DAL.DataModel;
    using sps.DAL.Entities;
    using sps.DAL.Repos.Base;
    using sps.DAL.Repos.Interfaces;
    public class {model}Repo : BaseRepo<{model}>, I{model}Repo
    {{
        public {model}Repo(DataContext dataContext) : base(dataContext)
        {{
        }}
    }}
}}
"""
# Generate the repository interface and implementation files for each model
for model in models:
    # Interface file
    interface_file_path = os.path.join(interfaces_dir, f'I{model}Repo.cs')
    with open(interface_file_path, 'w') as f:
        f.write(create_interface_content(model))
    # Repository implementation file
    repo_file_path = os.path.join(repos_dir, f'{model}Repo.cs')
    with open(repo_file_path, 'w') as f:
        f.write(create_repo_content(model))
print("Repository interface and implementation files generated successfully.")

I have asked the agent to now create my two contexts from my connection strings 


I am now in a situation where i would like to keep my progress.

	- Make a backend repo on github with this solution

I have commited a repo https://github.com/tundakk/sps-backend.git

now i try to keep track of my process by creating a branch each day with the date as name and commiting it to main/master with message of what i have been doing that day. 

this is probably very unescesarry and elaborate but id rather overkill

Friday 28-02


started creating configuration files with agent 

these are used by entity framework for mapping relationships and add attributes such as required. 



public class PlaceConfiguration : IEntityTypeConfiguration<Place>
    {
        public void Configure(EntityTypeBuilder<Place> builder)
        {
            builder.HasKey(p => p.Id);
            // Configure properties
            builder.Property(p => p.Name).IsRequired();
            builder.Property(p => p.PlaceNumber).IsRequired();
            builder.Property(p => p.Alias).IsRequired(false);
            // Configure relationships
            builder.HasMany(p => p.SupportingTeachers)
                .WithOne(st => st.Place)
                .HasForeignKey(st => st.PlacesId)
                .OnDelete(DeleteBehavior.SetNull);
        }
    }

the agent handled it well with the files. I will make a migration to "test" or check if the relationships are correct. 


NOW i will make the add-migration to the database.

hmm okay. The migration tool wants to build the solution. I cant do that right now. Too many unsolved problems. 



i will need to refactor all the business logic  and controllers. Then i can outcomment or removed the rest of the few issues after that. 

again i have 2 scripts for controllers and generic services 


import os
# Define the models for which you want to create controllers
models = [
    'Student',
'Education',
'SpsaCase',
'Period',
'Place',
'Diagnosis',
'EduCategory',
'StudentPayment',
'SupportingTeacher',
'OpkvalSupervision',
'EduStatus',
'EducationPeriodRate',
'TeacherPayment',
'SupportType',
]
# Directory to place the generated controller files (inside Implementations folder)
controllers_dir = './Controllers/Implementations'
# Ensure the directory exists
os.makedirs(controllers_dir, exist_ok=True)
# Function to create controller content
def create_controller_content(model):
    return f"""using sps.Api.Controllers.Base;
using sps.BLL.Services.Interfaces;
using sps.BLL.Infrastructure.Interfaces;
using sps.Domain.Model.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
namespace sps.API.Controllers.Implementations
{{
    /// <summary>
    /// {model}sController
    /// </summary>
    [ApiController]
    [Route("api/[controller]")]
    public class {model}sController : BaseController<{model}sController>
    {{
        private readonly I{model}Service _{model[0].lower() + model[1:]}Service;
        /// <summary>
        /// Initializes a new instance of the <see cref="{model}sController"/> class.
        /// </summary>
        /// <param name="{model[0].lower() + model[1:]}Service">The {model} service.</param>
        /// <param name="logger">The logger.</param>
        public {model}sController(I{model}Service {model[0].lower() + model[1:]}Service, ILogger<{model}sController> logger)
            : base(logger)
        {{
            _{model[0].lower() + model[1:]}Service = {model[0].lower() + model[1:]}Service;
        }}
        ///<inheritdoc/>
        [HttpGet]
        public IActionResult GetAll()
        {{
            try
            {{
                var response = _{model[0].lower() + model[1:]}Service.GetAll();
                if (!response.Success)
                {{
                    return BadRequest(response.Message);
                }}
                return Ok(response.Data);
            }}
            catch (Exception ex)
            {{
                return HandleError(ex);
            }}
        }}
        
        ///<inheritdoc/>
        [HttpGet("{id}")]
        public IActionResult GetById(Guid id)
        {{
            try
            {{
                var response = _{model[0].lower() + model[1:]}Service.GetById(id);
                if (!response.Success)
                {{
                    return BadRequest(response.Message);
                }}
                return Ok(response.Data);
            }}
            catch (Exception ex)
            {{
                return HandleError(ex);
            }}
        }}
        ///<inheritdoc/>
        [HttpPost]
        public IActionResult Insert([FromBody] {model}Model {model[0].lower() + model[1:]}Model)
        {{
            try
            {{
                var response = _{model[0].lower() + model[1:]}Service.Insert({model[0].lower() + model[1:]}Model);
                if (!response.Success)
                {{
                    return BadRequest(response.Message);
                }}
                if (response.Data == null)
                {{
                    return BadRequest("Failed to create the {model}");
                }}
                return CreatedAtAction(nameof(GetById), new {{ id = response.Data.Id }}, response.Data);
            }}
            catch (Exception ex)
            {{
                return HandleError(ex);
            }}
        }}
        
        ///<inheritdoc/>
        [HttpPut("{id}")]
        public IActionResult Update(Guid id, [FromBody] {model}Model {model[0].lower() + model[1:]}Model)
        {{
            try
            {{
                {model[0].lower() + model[1:]}Model.Id = id;
                var response = _{model[0].lower() + model[1:]}Service.Update({model[0].lower() + model[1:]}Model);
                if (!response.Success)
                {{
                    return BadRequest(response.Message);
                }}
                return Ok(response.Data);
            }}
            catch (Exception ex)
            {{
                return HandleError(ex);
            }}
        }}
        ///<inheritdoc/>
        [HttpDelete("{id}")]
        public IActionResult Delete(Guid id)
        {{
            try
            {{
                var response = _{model[0].lower() + model[1:]}Service.Delete(id);
                if (!response.Success)
                {{
                    return BadRequest(response.Message);
                }}
                return Ok(response.Data);
            }}
            catch (Exception ex)
            {{
                return HandleError(ex);
            }}
        }}
    }}
}}
"""
# Generate the controller files for each model
for model in models:
    controller_file_path = os.path.join(controllers_dir, f'{model}sController.cs')
    with open(controller_file_path, 'w') as f:
        f.write(create_controller_content(model))
print("Controller files generated successfully.")



I will now create the respective domain models to not directly call the entites in the DAL layer 

I just found out i should hash or encrypt cpr number so i will make this into a value object

i need to reverse it on the frontend so i can only encrypt. I will do the same with teacher email



using System;
using System.ComponentModel.DataAnnotations;
using System.Text.Json.Serialization;
namespace sps.Domain.Model.ValueObjects
{
    public class CPRNumber
    {
        [Required]
        [MaxLength(10)]
        public string Value { get; private set; }
        // Empty constructor required for EF Core
        protected CPRNumber() { }
        [JsonConstructor]
        public CPRNumber(string value)
        {
            if (string.IsNullOrEmpty(value))
                throw new ArgumentNullException(nameof(value), "CPR number cannot be null or empty");
            if (value.Length != 10)
                throw new ArgumentException("CPR number must be exactly 10 characters", nameof(value));
            Value = value;
        }
        // Implicit conversion to string for easier usage
        public static implicit operator string(CPRNumber cprNumber) => cprNumber.Value;
        // Implicit conversion from string for easier creation
        public static implicit operator CPRNumber(string value) => new CPRNumber(value);
        public override string ToString()
        {
            return Value;
        }
        public override bool Equals(object obj)
        {
            if (obj is CPRNumber other)
                return Value == other.Value;
            return false;
        }
        public override int GetHashCode()
        {
            return Value.GetHashCode();
        }
    }
}

I do the same with properties Email, AccountNumber, Comment, and student name property. 

since these are all string values, i have made a sensitivestring valueobject 



using System;
using System.ComponentModel.DataAnnotations;
using System.Text.Json.Serialization;
namespace sps.Domain.Model.ValueObjects
{
    public sealed class SensitiveString
    {
        private string _value;
        [Required]
        public string Value 
        { 
            get => _value;
            private init => _value = value ?? throw new ArgumentNullException(nameof(value));
        }
        // Empty constructor for EF Core
        private SensitiveString() { }
        [JsonConstructor]
        public SensitiveString(string value)
        {
            if (value == null)
                throw new ArgumentNullException(nameof(value));
            _value = value;
        }
        // For JSON serialization
        public override string ToString() => Value;
        // Required for proper value object equality
        public override bool Equals(object? obj)
        {
            if (obj is SensitiveString other)
                return Value == other.Value;
            return false;
        }
        public override int GetHashCode() => Value.GetHashCode();
        // For easier usage in code
        public static implicit operator string(SensitiveString sensitiveString) => sensitiveString.Value;
        public static explicit operator SensitiveString(string value) => new(value);
    }
}


i am now fixing namespaces and usings in my services that wasnt 100% correct from my scripts. 



	- I am fixing using statements for controllers. The scripts are not 100% correct so i am fixing references a little bit. Ex. It called non asyncronous methods on baseservice that doesnt exist anymore.


Monday 03

Now i ask for critique of my system and Claude sonnet 3.7 thinking, thinks i should extend/make my serviceresponse pattern more elaborate with methods and more propeties such validation errors, errorcodes and techincaldetails for development environement 
I really like the techincaldetails idea. I have struggled a bit with giving too much information back to the frontend in production. 


i have also added DTO'S for the frontend and test projects (i had these earlier, but the werent implemented correct.



Tuesday

All day has been refactoring and creating DTO logic. Its tedious and i lose oversight constantly. No fun… 


Whensday

I am building the frontend a little bit although the full functionality will come later. Ill just make some generic sites so i can share it more easily with the customer and get feedback of what each site should contain.


i made a mock login because i dont want to use the backend right now



Thursday

I am trying to outcomment my dtos because i lost oversight. Right now i will use the direct domain models and slowly implement the corret dtos

very tedous work now that the system is so large with so many entites. Like a days work fixing dataseed.cs and controllers

I have also comment property into a list of comments and save all comments to its own table with ids as relation to other tables. It feels nice to have most of the very sensitve data isolated.

i have made a commencontroller for this, a service, a repo and entity so theres a looot of added code



Monday 10-03

I used a lot of time trying to debug why my microsoft identity endpoints didnt show. I had implemented them twice, both in program.cs and the serviceextension. 
I found out when switiching from vs code insiders IDE to visual studio 2022. it made a breakpoint and said it was already implemented. Multiple hours of debugging. 


now in need  disable the /register endpoint, since i dont want this functionality when not logged in.

After much considertation i have chosen to NOT use the default endpoints. They seem too unsafe. Too much information is returned and its hard to extend or turn off some of them. 


i now use csrf tokens as well 


i have had many issue with turbopack so i changed to babel instead. Transpilers… i do not have HTAT much experience with react development / frontend edevelopment so this will probably be a long cumbersome development experience



Whensday

I used a lot of time make the frontend and backen communicate

CORS, no TSL, polices, correct format.
I can register new useres from the frontend now, but i am still struggling with the rest of my auth endpoints 








I started doing a pull request but there are sooo many artichteturial differences from when i made the main, so i might end up to just remove the old main and use the 28-02-2025 branch as thew main branch. Really bad practuce. WHOOPS this is the stuff of nightmares. Merge failures. I will just make the 28-02-2025 the new main branch. I have chosen not to write about gitflow and freature branches in my process report :'(


git checkout 28-02-2025
git push origin 28-02-2025:main --force
git checkout main
git fetch
git reset --hard origin/main


thrusday

I am rebuilding my frontend architecture a lot. I will use a clean architecture approach. 


## Architectural Structure

You've organized your frontend into clear layers following Clean Architecture principles:

1. **Presentation Layer** (`/app`)
   - Next.js UI components
   - Server actions as entry points
   - Focused on rendering and user interactions

2. **Interface Adapters** (`/interface-adapters/controllers`)
   - Controllers that validate input data
   - Transform data between layers
   - Handle errors at boundaries
   - Connect UI to business logic

3. **Application Layer** (`/application`)
   - Use cases that contain business logic
   - Ports (interfaces) that define service contracts
   - Independent of frameworks and UI concerns

4. **Domain Layer** (`/entities`, `/core`)
   - Models with Zod schemas for validation
   - Error types
   - Core business entities and rules

5. **Infrastructure Layer** (`/infrastructure`)
   - Concrete service implementations
   - External API communication
   - Storage mechanisms (local storage)


this helps a lot if i am allowed to integrate other services from their systems and not just their own system.
