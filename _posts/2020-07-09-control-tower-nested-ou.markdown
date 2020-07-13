Introduction

AWS provides Control Tower as an approach to organizing and configuring accounts.  Control Tower in turn takes advantage of several other AWS services, including Organizations.  This post is about an approach we wanted to take, what we found, and a solution involving some customisation of an AWS Solution.

AWS offers several approaches to organizing and configuring resources within Control Tower
  Individual account
  Organizational Unit (OU)
  Account and Region are Amazon's distinctions in keeping with their architecture, but OUs allow users additional freedom in organizing accounts in ways that don't fit those structures.

nested ous
  AWS limit of 5 levels
  OUs are hierarchical, all exist under the Organization root
  examples: Project1:Development, Project1:Test, Project1:Production, Project2:Development, Project2:Test, Project2:Production
  Notice in these examples that some OU names might be duplicated, distinguished only by their parent OU.
  

AWS Control Tower Customization Solution
  provides central tool to specify and deploy configuration and resources via SCPs and Cloudformation templates
  deployment may be specified by Account, Region, and OU

problem statement
  we are organizing our accounts into OUs, and nesting them
  This may require applying a restriction to a child OU without applying that restriction to the parent OU, and the documentation offers no example of doing this 
  I have confirmed that the Solution does not 'find' OUs by their individual name alone.  

progression to solution
  AWS pubilshes the source code and I examined that.  I found some mention of nested OUs, but the implementation seemed incomplete.
  The existing references suggested ':' as a delimimter character, so I've implemented that.

  talk through code, or just reference PR?
  

PR in AWS repo, fork it yourself, tell us about it


*** I don't see any equivalent mechanism for CloudFormation in the code. ***
