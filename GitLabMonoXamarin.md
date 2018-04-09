# GitLab on Linux with Mono, MonoGame, Xamarin.Android

- Prepare machine for Mono and Xamarin.Android (see other howto doc)
- Install GitLab
- Create project(s)
- Install GitLab Runner
- Register GitLab runner with GitLab
- Enable GitLab runner for projects

## Install MonoGame

According to: http://community.monogame.net/t/installing-monogame-3-6-on-linux/8811 run

wget http://www.monogame.net/releases/v3.6/monogame-sdk.run  
chmod +x monogame-sdk.run  
sudo ./monogame-sdk.run

### Build Dependencies for Xamarin Android in conjunction with Nuget

When getting monogame pipeline importers via nuget the following build target order is critical:

1. Get Nuget package with pipeline importers (otherwise pipeline tool will not find the importers)
2. Run MonoGame Pipeline tool (which actually creates files with build type "AndroidAsset" - see MonoGame.Content.Builder.targets Target BuildContent)
3. Run compile (which requires the files with build type "AndroidAsset" otherwise compile succeeds but APK does not contain any assets)



## Nuget for every project

- apt-get install nuget
- nuget update -self
- go to project directory
- create nuget file by calling "nuget spec"
- Creates Package.nuspec
- Edit Package.nuspec; Set version to 0.0.0
- If necessary add files element to Package.nuspec describing all files which should be included
- Build debug / release
- Run "nuget pack -Version 0.1.0" which should create a *.nupkg file 
- Run "nuget add *.nupkg -source /destination" to add package to local repository under /destination

## Nuget for Gitlab

- Add GitLab Runner to group which can access nuget repository directory
- As user gitlab-runner (and all other users accessing local repository): Add local rep to nuget with 'nuget sources Add -Name "SomeName" -Source /destination'
- Verify local repository is setup correctly "nuget sources" should list local repository (besides official)

## Gitlab for CI 

- Add .gitlab-ci.yml to root directory containing stages as required (build, test, deploy)
- Add "nuget restore" to before_script section
- Add artifacts to build section to save binaries compiled during build
- Add dependency to test and deploy stages to restore artifacts

## Security

I currently don't see how security for the keystore itself and its passwords is achieved. Options are GitLab security variables, jarsigner with password in a file or env variables. All things I suppose could be easily routed to the gitlab runner output.
There are already different thoughts about this but it seems there is no final conclusion on what to do. For example:
- https://gitlab.com/gitlab-org/gitlab-ce/issues/20826

Some thoughts:

Having a separate deployment (master, release) branch with an extra runner and protected variables. Only certain people are allowed to pull on that branch.

### Deploy to local nuget

Add lines to pack and add nuget to local repository (see "Nuget for every project") like:
- nuget pack -Version 0.$CI_PIPELINE_ID.0
- nuget add *.nupkg -source /destination
  
Note that $CI_PIPELINE_ID is replaced by GitLab. There are more pre defined GitLab viarables

## Deploy to Google Play

- tbd



