# APPLICATION-REQUIREMENTS.md
# General requirements 
## Main Goal: Allo Music Camps outsource the audition video management.

1. Generate Oracle Apex 26.1 app using Apexlang application for Camp admin management and music applicants help support and application instructions.
2. Application should have an Camp Admin section for Camp administrators to login and and review all the application videos. Videos are going to leave on external cloud storage. Camp Admins are related to the specific Camp and should be able to search/browse the submitted videos for the specific Camp only. All the submissions have to be devided into Application year categories.
3. Application admin section is required with the full access across all the application for every Camp. 
4. Tech support section is required with ability to Contact Tech support of the Site for the Camp administrators. 
5. Contact Camp Support section is required for Applicants to contact the Camp Management/Camp Adminiastartors. 
6. iOS application should be build to record the videos in secure way, so the video recording cannot be adjusted. iOS application should record the video and create a watermark that the video cannot be modified later. Every video has to have a signature or something like that to guarantee that orogonal video is not modified.
7. Android device application is required to follow the same requirements as the iOS application. 
8. MacOS and PC applications are required to follow the same standards as the iOS/Android device applications.
9. After video is recorded the info about the recording should be available at the Apex application.
10. Each applicant should have a limited number of recording attemps defined by the Camp administrators.
11. Camp administrators define the required repertuar for the applicants and the number of pieces/videos expected. The number of attempts applies to every piece.
12. camp administrators define the deadline date for the application. 
13. No applicant videos are visible to the Camp administrators until the whole application is submitted.
14. Applicant should be able to leave private comments on each recording attempt. Applicant;s notes are private and never visible to the Camp administrators even after the submission. 
15. Applicant should select the best attempt on every piece before submitting the application. 
16. When application is submitted, no new recordings should be allowed to the applicant. All the data becomes read only for the applicant after this point. 
17. After applicant submitts the application Camp admins should be able to see the submitted version of each piece of the application. Other versions should not be available to the camp administrators.
18. Simplified version of the Applicant submission process and selection of the best recording versions should be available on the iOS/Android applications.
19. The number of attepts left for each piece and status of the application should be highlighted on every page for the applicant.
20. Submission history of the previous year should be available to the applicant.
21. Camps should be able to see the previous year submission history for the same applicant.

## Apex Application requirements details
1. Security is the high priority. 
2. Applicans should not see the the applications of other applicants.
3. Camp Admins should not have access to the other casmp applications.

## iOS/Android/PC/MacOS requirements 
1. Video recording should be native with no slowness of device overload. 
2. Simplified version of the Applicant submission process and selection of the best recording versions should be available on the iOS/Android applications.
3. Authentication process should be the same as for the main Apex app.
4. Support of the external microfones should be supported by the iOS/Android devices.
5. Offline device mode should be supported with the dump of the recording when the internet is available.
6. Option ot offload recordings to the cloud server on wifi only should be available. 
7. Option to download recording for personal use or manual submission to the Camp should be availabe. Recording still shuold have the watermark and signature generated in this case.
8. The number of attempts left for each piece and status of the application should be highlighted on every page for the applicant.

## Additional future functionality to keep in mind for the future development
1. AI scan of the videos to identify strong/weak performance points and the general grade of the performance.
2. Camp can configure the template of AI scan to choose the skills to pay additional attention to.
3. Applicants can run a generic AI scan on the recording versions beofre submitting the application.
4. English is the default language, but Gernman, Franch and Italian languages should be supported in the future across the board.  
