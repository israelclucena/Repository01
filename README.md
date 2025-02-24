/* eslint-disable */
import {
  AfterViewInit, Component, ElementRef,
  Input, OnChanges, OnDestroy,
  OnInit, SimpleChanges, ViewChild,
  EventEmitter, Output, SecurityContext
} from '@angular/core';
import { DomSanitizer } from '@angular/platform-browser';
import { RoutingManagementService } from 'src/app/services/routing/routing-management.service';
import { UtilsApiService } from 'src/app/services/utils/utils.api.service';
import { MicrofrontDirective } from '@ng-darwin-wmf/microfront';
import { langMap } from 'src/app/shared/utils/langMap';
import { IFrameSettings } from 'src/app/interfaces/iframe.interface';

/**
 * @description Wraps and adds functionality to an iframe component
 */
@Component({
  selector: 'app-iframe',
  templateUrl: './iframe.component.html',
  styleUrls: ['./iframe.component.scss'],
})
export class IframeComponent extends MicrofrontDirective
  implements OnInit, OnChanges, AfterViewInit, OnDestroy {

  @Input() public url?: string;
  @Input() public ssoToken?: string;
  @Output() registreUserInput = new EventEmitter<string>();

  @ViewChild('iframe', { static: false }) iframe: ElementRef;
  public loading = true;
  public height?: any;
  public languageSelection: any = { value: null };
  public iframeElement: ElementRef | undefined;
  private iframeInterval: any;
  private failedAttempts = 0;
  private maxAttemptsIframeInterval = 120;
  private refreshInterval = 500;
  private ignoreSelectors = this.utilsApiService.getSelectorsToIgnore();
  private resizingSelectors: [IFrameSettings];
  private iframeLocation: any;

  /**
   * @description The Iframe Component constructor
   * @param routerManagment @see RoutingManagementService
   * @param sanitizer @see DomSanitizer
   */
  constructor(
    private routerManagment: RoutingManagementService,
    private sanitizer: DomSanitizer,
    private utilsApiService: UtilsApiService
  ) {
    super();
    this.routerManagment.getLastPath().subscribe((lastPath: string) => {
      //console.log('iframe.component.ts: iframe lastPath', lastPath);
    });
  }

  /**
   * @description Waits for the parameters to get parsed,
   * then updates the status and fetches the legacy URL
   * @returns A promise to await the ngOnInit for
   */
  override async ngOnInit(): Promise<void> {
    await super.ngOnInit();
    this.resizingSelectors = this.utilsApiService.getSelectorsForIframeHeightResizing();
  }

  /**
   * @description Emit an input message event when the input inputMessage is modified
   * @param changes The changes that occured
   */
  ngOnChanges(changes: SimpleChanges): void {
    //console.log('iframe.component.ts: changes iframe', changes);
    this.updateIframeHeight();
    this.eventsAvoidSessionExpired();
  }

  ngAfterViewInit(): void {
    if (this.url && this.ssoToken) {
      this.iframeElement = this.iframe;
      this.languageSelection = langMap(false, sessionStorage.getItem('selectedUserLanguage') ?? 'en');
      this.loadIframeContent();
    }
  }


  private loadIframeContent(): void {

    if (!this.url) {
      throw new Error('URL is not defined');
    }
    
    const actionURL = new URL(this.url);
    const encodedSsoToken = encodeURIComponent(this.ssoToken ?? '');
    const encodedLanguage = encodeURIComponent(this.languageSelection.value);
    const encodedDeviceInfo = encodeURIComponent(sessionStorage?.getItem('device-info-token') ?? '');

    const form = `
      <html>
        <form action="${actionURL.toString()}" id="iframe-form" class="iframe-form" method="post" target="iframe">
          <input type="hidden" name="auth" value="${encodedSsoToken}">
          <input type="hidden" name="language" value="${encodedLanguage}">
          <input type="hidden" name="x-santander-device" id="x-santander-device" value="">
        </form>
        <script>
          document.getElementById("x-santander-device").value = ${JSON.stringify(encodedDeviceInfo)};
          document.getElementById("iframe-form").submit();
        </script>
      </html>
    `;

    if (this.iframeElement) {
      this.iframeElement.nativeElement.srcdoc = this.sanitizer.sanitize(SecurityContext.HTML, this.sanitizer.bypassSecurityTrustHtml(form));
      this.watchIframeHeightChanges();
      this.eventsAvoidSessionExpired();
    }
  }

  /**
   * @description Function to set the height on iframe
   */
  private watchIframeHeightChanges(): void {
    if (!this.iframeInterval) {
      this.failedAttempts = 0;

      this.iframeInterval = setInterval(() => {
        this.updateIframeHeight();

        if (this.failedAttempts >= this.maxAttemptsIframeInterval) {
          clearInterval(this.iframeInterval);
        }
      }, this.refreshInterval);
    }
  }

  /**
   * @description Function to set the height on iframe
   */
  updateIframeHeight(): void {
    try {
      const shadowRootIframe = this.getShadowRootIframe();
      const innerIframe: any = shadowRootIframe?.contentDocument?.querySelector("iframe");
      let height = 0;

      const currentPath = window.location.pathname;
      let iFrameSettings = this.utilsApiService.getE2EIFrameHeightResizing().find(({url}) => currentPath.startsWith(url));

      if(!iFrameSettings) {

        if (!innerIframe) {
          throw Error('innerIframe not found');
        }

        const documentUrl = innerIframe?.contentWindow?.location?.href;
        if (!documentUrl) {
          throw Error('documentUrl not found');
        }
      
        iFrameSettings = this.getIFrameSettings(documentUrl);

        const rootComponent = innerIframe?.contentDocument?.querySelector(iFrameSettings?.selector) as HTMLElement;

        if (!rootComponent) {
          throw Error(`Parent div not found for selector: ${iFrameSettings?.selector}`);
        }

        height = Array.from(rootComponent.children)
          .filter(child => !this.isElementIgnored(child as HTMLElement))
          .map(child => child as HTMLElement)
          .reduce((total, child) => total + (child.offsetHeight ?? 0), 0);
      }

      // Set iframe height
      this.failedAttempts = 0;
      shadowRootIframe.style.height = height + (iFrameSettings?.heightIncrease ?? 0) + "px";
      
      if (this.height != height) {
        this.scrollToTop(shadowRootIframe);
      }
      this.height = height;

    } catch (error: any) {
      this.logError(`error in updateIframeHeight | ${error?.message}`, error);
    }
  }

  private getShadowRootIframe(): any {
    const legacyAppModule: any = document?.querySelector("legacy-app-module");

    if (!legacyAppModule) {
      throw new Error('legacyAppModule not found');
    }

    const shadowRoot: any = legacyAppModule?.shadowRoot?.querySelector("ng-component")?.shadowRoot;
    const shadowRootIframe: any = shadowRoot?.querySelector("app-iframe")?.querySelector("iframe");

    if (!shadowRootIframe) {
      throw new Error('shadowRootIframe not found');
    }

    return shadowRootIframe;
  }


  private getIFrameSettings(documentUrl: string): IFrameSettings {
    // Remove query parameters from the URL
    documentUrl = documentUrl.split('?')[0];
  
    // Try to find the corresponding iFrame settings
    const iFrameSettings = this.resizingSelectors.find(({ url }) => this.isExactMatch(documentUrl, url)) ?? {
      url: this.utilsApiService.getIFrameUrlDefault(),
      selector: this.utilsApiService.getIFrameHeightSelectorDefault(),
      heightIncrease: this.utilsApiService.getIFrameHeightIncreaseDefault()
    };
  
    return iFrameSettings;
  }

  private isExactMatch(documentUrl: string, url: string): boolean {
    return documentUrl === url;
  }

  private isElementIgnored(element: HTMLElement): boolean {
    return this.ignoreSelectors.some(selector => element.matches(selector));
  }

  private scrollToTop(shadowRootIframe?: any): void {
    const href = shadowRootIframe?.contentWindow?.location?.href;

    if (this.iframeLocation !== href) {
      this.iframeLocation = href;
      window.scrollTo({ top: 0, behavior: "smooth" });
    }
  }

  private eventsAvoidSessionExpired(): void {
    const shadowRootIframe = this.getShadowRootIframe();

    shadowRootIframe.contentWindow.addEventListener('mousemove', () => {
      this.registreUserInput.emit('mousemove');
    });
    shadowRootIframe.contentWindow.addEventListener('keydown', () => {
      this.registreUserInput.emit('keydown');
    });
    shadowRootIframe.contentWindow.addEventListener('click', () => {
      this.registreUserInput.emit('click');
    });

    const innerIframe: any = shadowRootIframe?.contentDocument?.querySelector("iframe");

    if (!innerIframe) {
      throw Error('innerIframe not found');
    }

    innerIframe.contentWindow.addEventListener('mousemove', () => {
      this.registreUserInput.emit('mousemove');
    });
    innerIframe.contentWindow.addEventListener('keydown', () => {
      this.registreUserInput.emit('keydown');
    });
    innerIframe.contentWindow.addEventListener('click', () => {
      this.registreUserInput.emit('click');
    });
  }

  private logError(message?: any, ...optionalParams: any[]): void {
    console.error(`iframe.component.ts: ${message}`, optionalParams);
    this.failedAttempts++;
  }

}
