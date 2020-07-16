# Introduction to InSpec Profiles

[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)

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
  
    
### inspec.yml
  
Each profile must have an inspec.yml file that defines the following information:  

Use `name` to specify a unique name for the profile. Required.  
Use `title` to specify a human-readable name for the profile.  
Use `maintainer` to specify the profile maintainer.  
Use `copyright` to specify the copyright holder.  
Use `copyright_email` to specify support contact information for the profile, typically an email address.  
Use `license` to specify the license for the profile.  
Use `summary` to specify a one line summary for the profile.  
Use `description` to specify a multiple line description of the profile.  
Use `version` to specify the profile version.  
Use `inspec_version` to place SemVer constraints on the version of Chef InSpec that the profile can run under.  
Use `supports` to specify a list of supported platform targets.  
Use `depends` to define a list of profiles on which this profile depends.  
Use `attributes` to define a list of attributes you can use in your controls.  
  
`name` is required; all other profile settings are optional. For example:  
```yml
name: ssh
title: Basic SSH
maintainer: Chef Software, Inc.
copyright: Chef Software, Inc.
copyright_email: support@chef.io
license: Proprietary, All rights reserved
summary: Verify that SSH Server and SSH Client are configured securely
version: 1.0.0
supports:
  - os-family: linux
depends:
  - name: profile
    path: ../path/to/profile
inspec_version: "~> 2.1"
```
  

### Verify Profiles
  
Use the inspec check command to verify the implementation of a profile:  
```bash
inspec check examples/profile
```
  

### Platform Support

Use the supports setting in the inspec.yml file to specify one (or more) platforms for which a profile is targeting.   
You can target all platforms in a single inspec.yml file:
```yml
name: ssh
supports:
  - platform-name: debian
  - platform-name: ubuntu
    release: 14.04
  - platform-family: redhat
  - platform: aws
```
  

### Profile Dependencies
A Chef InSpec profile can bring in the controls and custom resources from another Chef InSpec profile. Additionally, when inheriting the controls of another profile, a profile can skip or even modify those included controls.  
  
Before a profile can use controls from another profile, the to-be-included profile needs to be specified in the including profile’s `inspec.yml` file in the `depends` section. For each profile to be included, a location for the profile from where to be fetched and a name for the profile should be included. For example:  
```yml
depends:
- name: my-profile
  url: https://my.domain/path/to/profile.tgz
- name: profile-via-git
  url: https://github.com/myusername/myprofile-repo/archive/master.tar.gz
```

When you execute a local profile, the `inspec.yml` file will be read in order to source any profile dependencies. It will then cache the dependencies locally and generate an `inspec.lock` file.  
  
If you add or update dependencies in `inspec.yml`, dependencies may be re-vendored and the lockfile updated with `inspec vendor --overwrite`  

Once defined in the `inspec.yml`, controls from the included profiles can be used! Let’s look at some examples.  
  
With the `include_controls` command in a profile, all controls from the named profile will be executed every time the including profile is executed.  
  
```ruby
include_controls 'cis-windows-2016-baseline'
```
  
If there are only a handful of controls that should be executed from an included profile, it’s not necessarily to skip all the unneeded controls, or worse, copy/paste those controls bit-for-bit into your profile. Instead, use the `require_controls` command.
  
```ruby
require_controls 'cis-windows-2016-baseline' do
    control 'baseline-2' do
        impact 0,5
    end
end
```
  
### InSpec Inputs
Inputs are the “knobs” you can use to customize the behavior of Chef InSpec profiles. If a profile supports inputs, you can set the inputs in a variety of ways, allowing flexibility. Profiles that include other profiles can set inputs in the included profile, enabling a multi-layered approach to configuring profiles.  
  
Suppose you have a profile named `rock_critic`. In its profile metadata file (inspec.yml):  
```yml
# Optionally declare inputs in the profile metadata
# This lets you set up things like type checking, etc.
inputs:
- name: amplifier_max_volume
  description: How loud the amplifiers can go
  type: numeric
  # More options, including value: and priority: are possible here
```
  
In the profile’s control code:  
```yml
# Set a default value for an input.  This is optional.
input('amplifier_max_volume', value: 10)

control 'Big Rock Show' do
  describe input('amplifier_max_volume') do    # This line reads the value of the input
    it { should cmp 11 } # The UK'S LOUDEST BAND
  end
end
```
  

[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
