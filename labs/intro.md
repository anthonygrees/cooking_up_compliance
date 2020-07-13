# Introduction to InSpec

[Back to the Lab Index](../README.md)

### Profile Structure

A profile should have the following structure:

```bash
examples/profile
├── README.md
├── controls
│   ├── example.rb
│   └── control_etc.rb
├── libraries
│   └── extension.rb
|── files
│   └── extras.conf
└── inspec.yml
```
where:

`inspec.yml` includes the profile description (required)  
`controls` is the directory in which all tests are located (required)  
`libraries` is the directory in which all Chef InSpec resource extensions are located (optional)  
`files` is the directory with additional files that a profile can access (optional)  
`README.md` should be used to explain the profile, its scope, and usage
