# Requirements
1. Node.js - LTS preferred

   https://nodejs.org/en/

2. .NET Framework 4.6.2 Developer Pack

   https://dotnet.microsoft.com/download/dotnet-framework/net462/

3. .Net Core

   https://dotnet.microsoft.com/download

4. Visual Studio Core - Preferred editor

   https://code.visualstudio.com/Download

5. Power Apps CLI

   https://aka.ms/PowerAppsCLI

6. Configure your Power Platform environment

   Select your environment and then select: Settings | Product | Features

   Enable "Allow publishing of canvas apps with code components"

# Walk-through

Get your AccuWeather key. Apply for one here:
https://developer.accuweather.com/


1. md AccuWeatherPCF

2. cd AccuWeatherPCF

3. pac pcf init --namespace ScottishSummit --name GetMyWeather --template field

4. npm install

5. Modify you Manifest file as follows:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<manifest>
  <control namespace="ScottishSummit" constructor="GetMyWeather" version="0.0.1" display-name-key="GetMyWeather" description-key="GetMyWeather description" control-type="standard">
    <property name="apikey" display-name-key="apikey_Display_Key" description-key="apikey_Desc_Key" of-type="SingleLine.Text" usage="bound" required="true"/>
    <property name="location" display-name-key="location_Display_Key" description-key="location_Desc_Key" of-type="SingleLine.Text" usage="bound" required="true"/>
    <property name="weather" display-name-key="weather_Display_Key" description-key="weather_Desc_Key" of-type="Whole.None" usage="bound" required="true" />
    <resources>
      <code path="index.ts" order="1"/>
    </resources>
  </control>
</manifest>
```

6. Modify you index.ts file as follows:

```javascript
import { IInputs, IOutputs } from "./generated/ManifestTypes";

export class GetMyWeather implements ComponentFramework.StandardControl<IInputs, IOutputs> {
  private _apikey?: string;
  private _location?: string;
  private _weather: number = 0;
  private _notifyOutputChanged: () => void;

  /**
   * Empty constructor.
   */
  constructor() {}

  /**
   * Used to initialize the control instance. Controls can kick off remote server calls and other initialization actions here.
   * Data-set values are not initialized here, use updateView.
   * @param context The entire property bag available to control via Context Object; It contains values as set up by the customizer mapped to property names defined in the manifest, as well as utility functions.
   * @param notifyOutputChanged A callback method to alert the framework that the control has new outputs ready to be retrieved asynchronously.
   * @param state A piece of data that persists in one session for a single user. Can be set at any point in a controls life cycle by calling 'setControlState' in the Mode interface.
   * @param container If a control is marked control-type='standard', it will receive an empty div element within which it can render its content.
   */
  public init(
    context: ComponentFramework.Context<IInputs>,
    notifyOutputChanged: () => void,
    state: ComponentFramework.Dictionary,
    container: HTMLDivElement
  ) {
    this._location = context.parameters.location.formatted ? context.parameters.location.formatted : "";
    this._notifyOutputChanged = notifyOutputChanged;
  }

  /**
   * Called when any value in the property bag has changed. This includes field values, data-sets, global values such as container height and width, offline status, control metadata values such as label, visible, etc.
   * @param context The entire property bag available to control via Context Object; It contains values as set up by the customizer mapped to names defined in the manifest, as well as utility functions
   */
  public updateView(context: ComponentFramework.Context<IInputs>): void {
    if (this._location == context.parameters.location.formatted) return;

    this._apikey = context.parameters.apikey.formatted;
    this._location = context.parameters.location.formatted;

    fetch(`https://dataservice.accuweather.com/locations/v1/ES/search?apikey=${this._apikey}&q=${this._location}&language=es-es`, {
      method: "GET",
    })
      .then((res) => res.json())
      .then((res) => {
        fetch(`https://dataservice.accuweather.com/currentconditions/v1/${res[0].Key}?apikey=${this._apikey}&language=es-es`, {
          method: "GET",
        })
          .then((wres) => wres.json())
          .then((wres) => {
            this._weather = wres[0].Temperature.Metric.Value.toFixed(0);
            this._notifyOutputChanged();
          });
      });
  }

  /**
   * It is called by the framework prior to a control receiving new data.
   * @returns an object based on nomenclature defined in manifest, expecting object[s] for property marked as “bound” or “output”
   */
  public getOutputs(): IOutputs {
    return { weather: this._weather };
  }

  /**
   * Called when the control is to be removed from the DOM tree. Controls should use this call for cleanup.
   * i.e. cancelling any pending remote calls, removing listeners, etc.
   */
  public destroy(): void {
    // Add code to cleanup control if necessary
  }
}
```

7. npm run build

8. npm start

9. md AccuWeatherSolution

10. cd AccuWeatherSolution

11. pac solution init --publisher-name ScottishSummit --publisher-prefix ssp

12. pac solution add-reference --path ..

13. msbuild /t:restore

14. msbuild