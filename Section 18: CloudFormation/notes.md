# CloudFormation

- Infrastructure as Code
- declarative way of outlining your AWS Infrastructure, for any resources

- **Benefits**
    - No resources are manually created (excellent for control)
    - The code can be version controlled (git)
    - Changes to the infra are reviewed through code

    - **Cost**
        - Each resources within the stack is tagged with an identifier so you can easily see how much a stack costs you
        - You can estimate the costs of your resources using the CloudFormation template
        - Savings strategy: In Dev, you could automate the deletion of templates at 5PM and recreated at 8AM, safely

    - **Productivity**
        - Ability to destroy and re-create an infra on the cloud on the fly
        - Auto generation of Diagram for you templates
        - Declarative programming (no need to figure out ordering and orchestration)
    
    - S**eparation of concern**: create many stacks for many apps and many layers
        - VPC stacks
        - Network stacks
        - App stacks
    
    - **Don't re-invent the wheel**
        - Leverage existing templates on the web
        - Leverage the documentation

### How CloudFormation Works

- Templates have to be uploaded in S3 and then referenced in CloudFormation
- To update a template, we can't edit previous ones. We have to re-upload a new version of the template to AWS
- Stacks are identified by a name
- Deleting a stack deletes every single artifact that was created by CloudFormation

## Deploying CloudFormation Templates

- Manual Way:
    - Edit templates in the CloudFormation Designer
    - Using the console to input parameters, etc

- Automated Way:
    - Editing templates in a YAML file
    - Using the AWS CLI to deploy the templates
    - Recommended way when you fully want to automate your flow

### Building Blocks

- *Template Components*:
    1. **Resources: your AWS resources declared in the template (MANDATORY)**
    2. Parameters: the dynamic inputs for your template
    3. Mappings: the static variables for your template
    4. Outputs: References to what has been created
    5. Conditionals: List of conditions to perform resource creation
    6. Metadata

- *Template Helpers*:
    1. References
    2. Functions

## CloudFormation - Resources

- Resources are the core of your CloudFormation template
- They represent the different AWS Components that will be created and configured
- Resources are declared and can be referenced 

- AWS figures out creation, updates, and deletes of resources for us
- There are over 224 types
- Resource types identifiers are of the form: **AWS::aws-product-name::data-type-name**

- FAQ:
    - Q: Can I have a dynamic amount of resources
        A: No, everything in the CloudFormation template has to be declared
    - Q: Is every AWS Service supported?
        A: Almost. Only a select few niches are not. You can work around that using AWS Lambda Custom Resources 

## CloudFormation - Parameters

- Parameters are a way to provide inputs to your Templates

- They're important to know about if:
    - You want to *reuse* your templates across the company
    - Some inputs can not be determined ahead of time

- Parameters are extremely powerful, controlled and can prevent errors from happening in your templates thanks to types

- Parameters can be controlled by all these settings:
    - **Type**
        - string
        - number
        - CommaDelimitedList
        - List<Type>
        - AWS Parameter (to help catch invalid values - match against existing values in the AWS Account)
    - **Description**
    - **Constraints**
    - **ConstraintDescription (String)**
    - **Min/MaxLength**
    - **Min/MaxValue**
    - **Defaults**
    - **AllowedValues (array)**
    - **AllowedPattern (regex)**
    - **NoEche (bool)**

- **Reference a parameter using *Fn::Ref* function**
- The shorthand for this is ***!Ref***

- **Pseudo Parameters** - default parameters provided by AWS

## CloudFormation - Mappings

- Mappings are fixed variables within your CloudFormation Template
- handy for differentiating environements (dev/prod), regions (AWS regions), AMI types, etc
- All the values are hardcoded within the template

- **When to use mappings vs parameters**
    - Mappings are great when you know in advance all the values that can be taken and deduced from variables such as:
        - Region
        - AZ
        - AWS Account
        - Environment
        - Etc...
    - They allow safer control over the template

    - Use parameters when the values are really user specific

- We use ***Fn::FindInMap*** to return a named value from a specific key
- ***!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]***

## CloudFormation - Outputs

- declares *optional* values that we can import into other stacks (if you export them first)
- You can also view the outputs in the AWS Console or in using the AWS CLI
- useful for example, if you define a network CloudFormation, and output the variables such as VPC ID and your Subnet IDs
- it's the best way to perform some collaboration cross stack, as you let expert handle their own part of the stack
- You can't delete a CloudFormation stack if its outputs are being referenced by another CloudFormation stack

- **Cross Stack Reference**
    - Use the ***Fn::ImportValue*** function (shorthand ***!ImportValue***) to import an output from another stack


## CloudFormation - Conditions

- used to control the creation of resources or outputs based on condition
- Conditions can be whatever you want them to be, but common ones are:
    - Environment (dev/test/prod)
    - AWS Region
    - Any parameter value

- Each condition can be referenced by another condition, parameter value or mapping

- function list:
    - *Fn::And*
    - *Fn::Equals*
    - *Fn::If*
    - *Fn::Not*
    - *Fn::Or*

### CloudFormation Intrinsic Functions

- *Ref*
    - used to reference **parameters** and **resources (references the Resource ID)**
- *Fn::GetAtt*
    - return specific attributes of resources
- *Fn::FindInMap*
    - return a named value from a specific key in Mappings
- *Fn::ImportValue*
    - return value exported from another stack
- *Fn::Join*
    - join values with a delimiter
- *Fn::Sub*
    - shorthand for "substitute". 
    - String interpolation where variables are inserted with **${varName}**
- Conditional Functions (seen above)

# CloudFormation Rollbacks

- Stack Creation Fails:
    - Default: everything rolls back (gets deleted). We can look at logs
    - Option to disable rollback and troubleshoot what happened

- Stack Update Fails:
    - The stack automatically rolls back to the previous known working state
    - Ability to see in the log what happened and error messages

# CloudFormation - Stack Notifications

- Send Stack events to SNS Topic (Email, Lambda)
- Enable SNS Integration using Stack Options

# CloudFormation - ChangeSets, Nested Stacks & Stack Sets

## ChangeSets

- When you update a stack, you need to know what changes before it happens for greater confidence
- ChangeSets won't say if the update will be successful

## Nested Stacks

- stacks part of other stacks
- Allow you to isolate repeated patterns / common components in separate stacks and call them from other stacks
- Ex:
    - Load Balancer configuration that is re-used
    - Security Group that is reused
- Nested Stacks are considered best practice
- To update a nested stack, always update the parent (root stack)

- **Cross vs Nested Stack:**
    - **Cross Stack**
        - helpful when stacks have different lifecycles
        - actual resource instances are reused
        - Use Outputs Export and Fn::ImportValue
        - When you need to pass export values to many stacks
    - **Nested Stack**
        - Helpful when components must be re-used
        - only configuration is reused
        - Ex: re-use how to properly configure an ALB
        - The nested stack only is important to the higher level stack (it's not shared)


## StackSets

- CRUD across **multiple accounts and regions** with a single operation
- Admin account to create StackSets
- Trusted accounts to create, update, delete stack instances from StackSets
- When you update a StackSet, all associated stack instances are updated throughout all accounts and regions

# CloudFormation Drift

- **Drift** is manual configuration changes over time that may be unwanted
- We can use **CloudFormation Drift** to know if resources have drifted

# CloudFormation Stack Policies

- During a CloudFormation Stack update, all update actions are allowd on all resources (default)
- **A Stack Policy is a JSON document that defines the update actions that are allowed on specific resources during Stack updates**
- used to protect resources from unintended updates
- When you set a Stack Policy, all resources in the Stack are PROTECTED by default
- Specify an explicit ALLOW for the resources you want to be allowed to be updated

