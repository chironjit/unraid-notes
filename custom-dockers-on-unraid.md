# Adding custom dockers on Unraid from Docker hub
Guide on adding custom docker (distilled from the video below). Using Seafile as an example

## Get docker link
Head to dockerhub and get the docker tag:
`seafileltd/seafile:latest`

## Add container
1. Select `Add container` on docker tab in Unraid
2. In `Basic View`, add:
    Name: Choose any name you would like
    Overview (optional)
    Additional requirements(optional)
    Repository: `seafileltd/seafile:latest`
    Network type (leave as default)
    Console shell command (leave as default)
    Privileged (leave turned off unless you know what you're doing)
    





Source:
This summary has been distilled down from this video. Really appreciative of the author for this specific guide

[How To Deploy Docker Containers in UNRAID (including from Private Docker Registries)](https://www.youtube.com/watch?v=SZkfOehWu_s)