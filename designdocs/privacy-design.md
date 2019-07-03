## Note
This document was used as background for the [Privacy and Security Explainer](../privacy-security-explainer.md) and has been superseded by that explainer.

## Summary
This design adopts the [requirements of the Generic Sensors API](https://www.w3.org/TR/generic-sensor/#concepts-can-expose-sensor-readings) when exposing data based on sensors.

Further, when allowing access to any sensor-based data (ex. poses) or user configurable data (ex. IPD, bounds), the user agent must either apply all necessary mitigations to ensure user privacy, or obtain user consent at session creation in response to requestSession().

In some scenarios, mitigations may not be sufficient to ensure user privacy. In those situations, user consent is required. For example, user consent is always required when exposing data that might allow a site to profile the user. A summary of scenarios where user consent is required can be found under [User Communication](#user-communication).

Further, some mitigations are always required even if user consent is obtained (ex. focused and visible is always required before exposing pose data).

User consent must be requested only in response to requestSession(). It is recommended that consent last as long as the browsing context. User consent cannot be obtained during an active session, as there is no known approach for a trusted user interface that works consistently across all platforms during an active session.

When creating a session, all privacy-sensitive data that could be exposed during that session must be considered when determining whether or not user consent is required. If user consent is not granted for a particular data type, then either:
*   Other mitigations for that data type must be applied to ensure user privacy, or
*   Any feature that would expose that data type must be disabled, or
*   The session request must be rejected.

Not all sessions require user consent, such as:
*   Sessions that do not include any sensor-based or user-configured data;
*   Sessions that fully mitigate sensor-based privacy concerns (ex. pose quantization and position bounds) and which do not include user-configured data that could be used for profiling.

## Note
There is no distinction between different WebXR modes in this document. The requirements to expose any given data type are the same whether the session is inline, immersive, or otherwise. If a user agent wants to avoid user consent for a particular type of session, the user agent must meet the mitigation requirements to expose all of the data types available in that session.

However, not all modes and platforms support all types of data, and some mitigation strategies that work in one mode may not work in other modes or on other platforms. As a result, the actual requirements for each mode and platform will vary. For example, if in an inline session a user agent only supports the [viewer](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-viewer) reference space, does not support other forms of sensor-based data (e.g. from a 6DOF controller), and does not support multiple views, then all privacy requirements have been met and user consent is not required for that session. In contrast, an immersive session supporting the [unbounded](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-unbounded) reference space type with a native origin that is tracked using sensors would have different requirements for user consent and privacy mitigations. 


## Privacy Requirements
### Requirements by feature

The user agent must ensure the following requirements are met before enabling the corresponding WebXR feature or allowing access to associated data. This table is intended as a summary, more detail on each requirement is available in subsequent sections.

_Note: Some features have more than one requirement and appear in multiple rows._

<table>
   <tr>
      <td><b>Feature</b></td>
      <td><b>Requirements</b></td>
      <td><b>Why</b></td>
   </tr>
   <tr>
      <td>Any native origin or combination of native origins that allow calculation of real-world viewer movement.</td>
      <td>
Position Limiting for all pose data generated by all XRSpaces in the session
<br/><br/>
OR
<br/><br/>
User consent.
      </td>
   <td>The site may infer the user’s geographic location from unbounded pose data.</td>
   </tr>
   <tr>
   <td>Pose data</td>
   <td>
Same Origin
<br/><br/>
AND
<br/><br/>       
Focused and Visible
   </td>
   <td>Prevents Input Sniffing</td>
   </tr>
   <tr>
   <td>Access to data generated by sensors</td>
   <td>
Generic Sensor API requirements
<br/><br/>
AND
<br/><br/>
Feature policies for underlying sensors are respected.
   </td>
   <td>Prevents access to sensor data outside the scope of the requirements for the lower-level sensors.</td>
   </tr>
   <tr>
      <td>Viewer Height based on real world height</td>
      <td>User consent</td>
      <td>Site may profile user</td>
   </tr>
   <tr>
      <td>Emulated Viewer Height set by user</td>
      <td>Rounding</td>
      <td>Site may fingerprint device</td>
   </tr>
   <tr>
     <td>boundsGeometry</td>
     <td>Rounding</td>
     <td>Site may fingerprint device</td>
   </tr>
   <tr>
     <td>Configurable IPD as exposed by multiple Views</td>
     <td>
Rounding
<br/><br/>
AND
<br/><br/>
User consent
     </td>
     <td>
Site may profile user.
<br/><br/>
Site may fingerprint device based upon specific IPD distance.
     </td>
   </tr>
   <tr>
   <td>Multiple displays whose positions and orientations have been configured by the user (e.g. a CAVE system)</td>
   <td>Rounding</td>
   <td>Site may fingerprint device</td>
   </tr>
   <tr>
   <td>XRPose data generated by sensors</td>
   <td>
Quantization
<br/><br/>
OR
<br/><br/>
User consent
   </td>
   <td>Site may fingerprint device</td>
   </tr>
<table>
   
### Conditions to create objects and expose data
#### XRSession Creation

The user agent must verify that all mandatory conditions are satisfied to ensure it can create an [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) and expose [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) data in response to a call to [XRRequestSession](https://immersive-web.github.io/webxr/#dom-xr-requestsession) within a given [active document](https://html.spec.whatwg.org/multipage/browsers.html#active-document).

The **_mandatory conditions_** for [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) creation are the following:
*   The document is a [responsible document](https://html.spec.whatwg.org/multipage/webappapis.html#responsible-document) of a [secure context](https://w3c.github.io/webappsec-secure-contexts/#secure-contexts).
*   [Currently focused area](https://html.spec.whatwg.org/multipage/interaction.html#currently-focused-area-of-a-top-level-browsing-context) belongs to a document whose origin is [same origin-domain](https://html.spec.whatwg.org/multipage/origin.html#same-origin-domain) with the origin of the given [active document](https://html.spec.whatwg.org/multipage/browsers.html#active-document).
*   The document is [allowed to use](https://wicg.github.io/feature-policy/#allowed-to-use) all the [policy-controlled features](https://wicg.github.io/feature-policy/#policy-controlled-feature) associated with the given [sensor types](https://www.w3.org/TR/2018/CR-generic-sensor-20180320/#sensor-type) of sensors used to generate data available in the [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface). This includes sensors used to generate [XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose), [XRPose](https://immersive-web.github.io/webxr/#xrpose-interface) and [XRBoundedReferenceSpace](https://immersive-web.github.io/webxr/#xrboundedreferencespace-interface) data, and any other data generated by sensors. _Note: The intent of this requirement is to ensure that WebXR does not allow a site to isolate and extract sensor data that would otherwise be blocked by feature policy._
*   User consent is required before creating an [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) when the [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) will support the creation and use of [bounded-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-bounded-floor) or [unbounded](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-unbounded) reference space types.
*   User consent is required before creating an [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) when the [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) will support the creation and use of a [local-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-local-floor) reference space type, and the floor level will reflect the real-world location of the user’s floor.
*   User consent is required before creating an [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) on a device with configurable interpupillary distance (IPD), if subsequent calls to [XRFrame.getViewerPose()](https://immersive-web.github.io/webxr/#dom-xrframe-getviewerpose) will return [XRView.transform](https://immersive-web.github.io/webxr/#dom-xrview-transform)s that can be used to compute the configured IPD.
*   User consent is required before creating an [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) if the user agent does not otherwise [mitigate](#providing-xrviewerpose-data) [sensor fingerprinting threats](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-fingerprinting) through data access to [XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose-interface) or [XRPose](https://immersive-web.github.io/webxr/#xrpose-interface).

If user consent is required, it must be obtained as part of the [XRRequestSession](https://immersive-web.github.io/webxr/#dom-xr-requestsession) algorithm before the new [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) object is created. If the required consent is not obtained, the user agent MUST [reject](https://www.w3.org/2001/tag/doc/promises-guide/#reject-promise) the promise with a "[SecurityError](https://heycam.github.io/webidl/#securityerror)" [DOMException](https://heycam.github.io/webidl/#idl-DOMException).

```
Editor Note: The paragraph above suggests specific changes to the XRSession algorithm language.
```

#### Providing XRViewerPose data
The user agent must verify that all mandatory conditions are satisfied to ensure it can expose [XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose) data in response to each call to [getViewerPose](https://immersive-web.github.io/webxr/#dom-xrframe-getviewerpose) within a given [active document](https://html.spec.whatwg.org/multipage/browsers.html#active-document).

The **_mandatory conditions_** for exposing XRViewerPose data are the following:
*   [Visibility state](https://www.w3.org/TR/page-visibility-2/#dom-visibilitystate) of the document is "visible".
*   [Currently focused area](https://html.spec.whatwg.org/multipage/interaction.html#currently-focused-area-of-a-top-level-browsing-context) belongs to a document whose origin is [same origin-domain](https://html.spec.whatwg.org/multipage/origin.html#same-origin-domain) with the origin of the given [active document](https://html.spec.whatwg.org/multipage/browsers.html#active-document).
*   The document is [allowed to use](https://wicg.github.io/feature-policy/#allowed-to-use) all the [policy-controlled features](https://wicg.github.io/feature-policy/#policy-controlled-feature) associated with the given [sensor types](https://www.w3.org/TR/2018/CR-generic-sensor-20180320/#sensor-type) of sensors used to generate [XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose) data.
*   If the reference space is of type [local](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-local), [local-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-local-floor), or [bounded-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-bounded-floor), then the [XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose) data must be limited to a region approximately the size of a large room, to prevent such data from being used to [determine the user’s real-world location](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-location). Specific bounds are at the discretion of the user agent, but 30m x 30m is suggested as a reasonable limit. These bounds may be affected by previously created reference spaces.
*   If [XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose) data is generated by sensors, at least one of the following conditions must be met to mitigate [sensor fingerprinting threats](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-fingerprinting); the user agent may choose the approach that provides the best user experience:
    *   User consent must have been obtained, _or_
    *   Any [XRView](https://immersive-web.github.io/webxr/#xrview) transform data generated by sensors must be [quantized](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#rounding-quantization-or-fuzzing) to prevent sensor fingerprinting, _or_
    *   The user agent must otherwise ensure that the underlying device sensors are not susceptible to sensor fingerprinting.
*   User consent must have been obtained on devices with configurable interpupillary distance before creating an [XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose-interface) that will return an [XRView](https://immersive-web.github.io/webxr/#xrview) for each eye.
*   If the device supports configurable or factory-calibrated interpupillary distance that may vary from device to device, then the [XRView](https://immersive-web.github.io/webxr/#xrview) transform data must be rounded to prevent fingerprinting. Specific precision for rounding is at the discretion of the user agent.
*   If the [XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose-interface) will include multiple [XRView](https://immersive-web.github.io/webxr/#xrview)s for displays whose positions and orientations have been configured by the user (e.g. in a CAVE) then the [XRView](https://immersive-web.github.io/webxr/#xrview) transform data must be rounded to prevent fingerprinting.

#### Providing XRPose data
The user agent must verify that all mandatory conditions are satisfied to ensure it can expose [XRPose](https://immersive-web.github.io/webxr/#xrpose) data in response to each call to [getPose](https://immersive-web.github.io/webxr/#dom-xrframe-getpose) within a given [active document](https://html.spec.whatwg.org/multipage/browsers.html#active-document).

The **_mandatory conditions_** for exposing [XRPose](https://immersive-web.github.io/webxr/#xrpose) data are the following:
*   [Visibility state](https://www.w3.org/TR/page-visibility-2/#dom-visibilitystate) of the document is "visible".
*   [Currently focused area](https://html.spec.whatwg.org/multipage/interaction.html#currently-focused-area-of-a-top-level-browsing-context) belongs to a document whose origin is [same origin-domain](https://html.spec.whatwg.org/multipage/origin.html#same-origin-domain) with the origin of the given [active document](https://html.spec.whatwg.org/multipage/browsers.html#active-document).
*   The document is [allowed to use](https://wicg.github.io/feature-policy/#allowed-to-use) all the [policy-controlled features](https://wicg.github.io/feature-policy/#policy-controlled-feature) associated with the given [sensor types](https://www.w3.org/TR/2018/CR-generic-sensor-20180320/#sensor-type) of sensors used to generate the inputs [getPose](https://immersive-web.github.io/webxr/#dom-xrframe-getpose).
*   If an input to [getPose](https://immersive-web.github.io/webxr/#dom-xrframe-getpose) has a native origin that tracks a real-world location (e.g. a 6DOF controller) then the resulting [XRPose](https://immersive-web.github.io/webxr/#xrpose) data must be bounded by the same [position limiting](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#position-limiting) as for [getViewerPose](https://immersive-web.github.io/webxr/#dom-xrframe-getviewerpose), to prevent such data from being used to [determine the user’s real-world location](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-location). _Note: If [getViewerPose](https://immersive-web.github.io/webxr/#dom-xrframe-getviewerpose) is performed from a [viewer](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-viewer) reference space, then this position limiting requirement *does* still apply._
*   If an input to [getPose](https://immersive-web.github.io/webxr/#dom-xrframe-getpose) is a reference space of type [local](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-local), [local-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-local-floor), or [bounded-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-bounded-floor), then the resulting [XRPose](https://immersive-web.github.io/webxr/#xrpose) data must be bounded by the same [position limiting](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#position-limiting) for that reference space as [getViewerPose](https://immersive-web.github.io/webxr/#dom-xrframe-getviewerpose), to prevent such data from being used to [determine the user’s real-world location](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-location). _See: [Providing XRViewerPose data](#xrviewerpose)._
*   If an input to [getPose](https://immersive-web.github.io/webxr/#dom-xrframe-getpose) includes data generated by sensors, at least one of the following conditions must be met to mitigate [sensor fingerprinting threats](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-fingerprinting); the user agent may choose the approach that provides the best user experience:
    *   User consent must have been obtained, _or_
    *   Any transform data generated by sensors must be [quantized](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#rounding-quantization-or-fuzzing) to prevent sensor fingerprinting, _or_
    *   The user agent must otherwise ensure that the underlying device sensors are not susceptible to sensor fingerprinting.

#### Creating XRReferenceSpace
The user agent must verify that all mandatory conditions are satisfied to ensure it can create an [XRReferenceSpace](https://immersive-web.github.io/webxr/#xrreferencespace-interface) and expose [XRReferenceSpace](https://immersive-web.github.io/webxr/#xrreferencespace-interface) data within a given [active document](https://html.spec.whatwg.org/multipage/browsers.html#active-document).

The **_mandatory conditions_** for creating an XRReferenceSpace are the following:
*   User consent must have been obtained before creating a [bounded-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-bounded-floor) or [unbounded](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-unbounded) reference space. 
*   User consent is required before creating a [local-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-local-floor) reference space type, if the floor level will reflect the real-world location of the user’s floor.
*   When creating a [local-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-local-floor) or [bounded-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-bounded-floor) reference space type, if the origin of the reference space reflects the real-world location of the user’s floor, or if the floor level is emulated and set by the user to a non-default value, then the coordinates of the origin must be [rounded sufficiently](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#rounding-quantization-or-fuzzing) to prevent fingerprinting using floor level. Rounding to the nearest 1cm is suggested.
*   When creating a [bounded-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-bounded-floor) reference space the [boundsGeometry](https://immersive-web.github.io/webxr/#dom-xrboundedreferencespace-boundsgeometry) must be rounded sufficiently to prevent fingerprinting the user’s bounds. The region represented by the rounded bounds geometry must be a subset of the original bounds. Rounding to the nearest 5cm is suggested.
*   Creating multiple [bounded-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-bounded-floor), [local-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-local-floor) or [local](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-local) reference spaces during the same session that have different [effective origins](https://immersive-web.github.io/webxr/#xrspace-effective-origin) can expose the same threat vectors present in an [unbounded](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-unbounded) reference space. User agents must either:
    *   Require the same user consent as for [unbounded](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-unbounded) reference spaces, before allowing creation of multiple reference spaces with different [effective origins](https://immersive-web.github.io/webxr/#xrspace-effective-origin) in the same session, _or_
    *   Enforce the same [position limiting](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#position-limiting) bounds for [XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose-interface) data within all those sessions, and require that all [effective origins](https://immersive-web.github.io/webxr/#xrspace-effective-origin) fall within those bounds.

### Obtaining User Consent
Acquiring user consent is a mandatory condition for some types of object creation and data access. In addition to general [considerations](https://github.com/immersive-web/privacy-and-security/blob/master/EXPLAINER.md#considerations) for obtaining user consent, the following considerations are specific to the WebXR Device API.

#### Lifetime of Consent
The following guidelines are suggested:
*   User consent should only be considered valid for the [browsing context](https://www.w3.org/TR/2009/WD-html5-20090423/browsers.html#browsing-context) within which it was obtained.
*   Once a specific consent is obtained for a specific [origin](https://www.w3.org/TR/2009/WD-html5-20090423/browsers.html#origin-0) in a [browsing context](https://www.w3.org/TR/2009/WD-html5-20090423/browsers.html#browsing-context), that [origin](https://www.w3.org/TR/2009/WD-html5-20090423/browsers.html#origin-0) does not need to obtain that specific consent again during the lifetime of that [browsing context](https://www.w3.org/TR/2009/WD-html5-20090423/browsers.html#browsing-context). Specifically, if multiple same-origin [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) objects are created in a [browsing context](https://www.w3.org/TR/2009/WD-html5-20090423/browsers.html#browsing-context), and all require the same user consent, then consent should only need to be obtained once.
*   The user agent must ensure that all mandatory conditions for user consent are met before creating any [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) object. As a result, the user agent may be required to ask for user consent multiple times in a [browsing context](https://www.w3.org/TR/2009/WD-html5-20090423/browsers.html#browsing-context) if it is creating multiple [XRSession](https://immersive-web.github.io/webxr/#xrsession-interface) objects each with different mandatory conditions for user consent.

#### User Communication
The judgement on how to communicate to the user the known threats is up to the implementer. It is suggested that the following threat vectors be communicated to the user at the time of session creation:

<table>
  <tr>
   <td><strong>User consent is required if...</strong>
   </td>
   <td><strong>Why user consent is required</strong>
   </td>
  </tr>
  <tr>
   <td>XRSession can create an <a href="https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-unbounded">unbounded</a> reference space.
   </td>
   <td>Site may be able to determine user’s specific geographic <a href="https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-location">location</a>, and may be able to perform gait analysis, allowing user <a href="https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-profiling">profiling</a> and <a href="https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-fingerprinting">fingerprinting</a>.
   </td>
  </tr>
   <tr>
   <td>XRSession can create an <a href="https://immersive-web.github.io/webxr/#xrspace-interface">XRSpace</a> and <a href="https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#position-limiting">position limiting</a> is not applied.
   </td>
   <td>Site may be able to determine user’s specific geographic <a href="https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-location">location</a>.
   </td>
  </tr>
 <tr>
   <td>XRSession can create multiple <a href="https://immersive-web.github.io/webxr/#xrspace-interface">XRSpace</a> objects with different <a href="https://immersive-web.github.io/webxr/#xrspace-native-origin">native origins</a> and suitable <a href="https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#position-limiting">position limiting</a> is not applied.
   </td>
   <td>Site may be able to determine user’s specific geographic <a href="https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-location">location</a>.
   </td>
  </tr>  
  <tr>
   <td>XRSession can create a <a href="https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-local-floor">local-floor</a> or <a href="https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-bounded-floor">bounded-floor</a> reference space, where the origin of the reference space reflects the real-world location of the user’s floor.
   </td>
   <td>Site may be able to infer the user’s height and may be able to perform gait analysis, allowing user <a href="https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-profiling">profiling</a> and <a href="https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-fingerprinting">fingerprinting</a>.
   </td>
  </tr>
  <tr>
   <td>XRSession can provide <a href="https://immersive-web.github.io/webxr/#xrviewerpose">XRViewerPose</a> data that is not quantized, and underlying device sensors may be susceptible to fingerprinting.
   </td>
   <td>Site may be able to perform user <a href="https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-fingerprinting">fingerprinting</a>.
   </td>
  </tr>
  <tr>
   <td>On a device with configurable interpupillary distance, XRSession may create an <a href="https://immersive-web.github.io/webxr/#xrviewerpose-interface">XRViewerPose</a> that will return an <a href="https://immersive-web.github.io/webxr/#xrview">XRView</a> for each eye
   </td>
   <td>Site access to IPD data may allow user <a href="https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-profiling">profiling</a> and <a href="https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-fingerprinting">fingerprinting</a>.
   </td>
  </tr>
</table>

#### Camera Data and XRSession Data
The combination of camera data (e.g. using [getUserMedia](https://www.w3.org/TR/mediacapture-streams/#dom-mediadevices-getusermedia())) with [XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose) data that is based on real-world viewer position and orientation may expose threat vectors related to [real-world geometry](https://github.com/immersive-web/privacy-and-security/blob/master/EXPLAINER.md#real-world-geometry) that are not present when only camera data is available.

Such threat vectors assume that both types of data are available within a sufficiently short time interval that, given a camera frame, the viewer pose can be known at the time the frame was captured.

It is suggested that the user agent either prevent access to both types of data on the same [origin](https://www.w3.org/TR/2009/WD-html5-20090423/browsers.html#origin-0) within a short time interval (e.g. 2 seconds), or inform the user of the threat vectors and obtain user consent before making both types of data available.

### Private Browsing Modes
User agents may support a mode (e.g., private browsing) of operation intended to preserve user anonymity and/or ensure records of browsing activity are not persisted on the client.

There are <span style="text-decoration:underline;">no</span> additional requirements for such modes, as there is no persistent data or unique user identifier data generated by the WebXR Device API.

## Background
### Threats and Mitigations
The following threats, and associated mitigations, are the basis of the above privacy requirements.

#### Pose Data
[XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose) or [XRPose](https://immersive-web.github.io/webxr/#xrpose-interface) might be used for [input sniffing](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#reading-inputs-or-input-sniffing). [Same Origin](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#same-origin-or-single-origin-only) and [Focused and Visible](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#focused-and-visible) mitigations address this threat.

[XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose) or [XRPose](https://immersive-web.github.io/webxr/#xrpose-interface) might be used to [identify the user’s location](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-location). [Position Limiting](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#position-limiting) addresses this threat for [bounded-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-bounded-floor), [local-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-local-floor) or [local](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-local) reference spaces. User consent is required for [unbounded](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-unbounded) reference spaces, or if the user agent allows the creation of multiple reference spaces in the same session that have different [position limiting](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#position-limiting) bounds.

[XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose) or [XRPose](https://immersive-web.github.io/webxr/#xrpose-interface) might be used for [gaze tracking](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#gaze-tracking). [Same Origin](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#same-origin-or-single-origin-only) and [Focused and Visible](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#focused-and-visible) mitigations address this threat.

[XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose) or [XRPose](https://immersive-web.github.io/webxr/#xrpose-interface) may allow [user fingerprinting](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-fingerprinting) when based on sensor data. The user agent may use [quantization](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#rounding-quantization-or-fuzzing) to address this threat, where data quantization will not affect the user experience. Otherwise, user consent is required.

[XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose-interface) may allow [user fingerprinting](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-fingerprinting) or [user profiling](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-profiling) on devices with configurable interpupillary distance. In this case, user consent is required.

[XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose-interface) may allow [user fingerprinting](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-fingerprinting) in situations where the user has configured several display orientations and positions that are represented as multiple [XRView](https://immersive-web.github.io/webxr/#xrview)s (for example, a CAVE system). [Rounding](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#rounding-quantization-or-fuzzing) helps alleviate this threat.

#### Reference Space Data
Viewer height may be used for [user profiling](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-profiling). User consent is required in situations where this may be determined by real-world floor level.

Viewer height may be used for [user fingerprinting](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-fingerprinting), particularly in situations where viewer height is emulated and set by the user. [Rounding](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#rounding-quantization-or-fuzzing) addresses this threat.

Within a [bounded-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-bounded-floor) reference space, [boundsGeometry](https://immersive-web.github.io/webxr/#dom-xrboundedreferencespace-boundsgeometry) may be used for [user fingerprinting](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-fingerprinting). [Rounding](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#rounding-quantization-or-fuzzing) addresses this threat.

#### User Consent
Users may become [confused or fatigued](https://github.com/immersive-web/privacy-and-security/blob/master/EXPLAINER.md#permissions) if prompted excessively for user consent during a session. This is addressed by requiring user consent once, at time of session creation.

### Unaddressed
If the active document has access to the camera on a passthrough device, [XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose-interface) may allow [gaze tracking](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#gaze-tracking) of the real world. This potential threat is not addressed by these requirements.

If the active document can read back pixels outside its own rendering context (e.g. during a screen recording), then [XRViewerPose](https://immersive-web.github.io/webxr/#xrviewerpose-interface) may allow [gaze tracking](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#gaze-tracking). This potential threat is not addressed by these requirements.

Within a [bounded-floor](https://immersive-web.github.io/webxr/#dom-xrreferencespacetype-bounded-floor) reference space, [boundsGeometry](https://immersive-web.github.io/webxr/#dom-xrboundedreferencespace-boundsgeometry) might in theory be used for [user profiling](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-profiling). This potential threat is not addressed by these requirements.

Data generated by [XRInputSource](https://immersive-web.github.io/webxr/#xrinputsource-interface) might in theory be used for [user profiling](https://github.com/immersive-web/privacy-and-security/blob/master/POSE-AND-ENVIRONMENT.md#user-profiling), for example measuring the length of a user’s arm. This potential threat is not addressed by these requirements.