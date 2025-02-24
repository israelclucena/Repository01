import { ComponentFixture, TestBed } from '@angular/core/testing';
import { IframeComponent } from './iframe.component';
import { UtilsApiService } from 'src/app/services/utils/utils.api.service';
import { RoutingManagementService } from 'src/app/services/routing/routing-management.service';
import { DomSanitizer } from '@angular/platform-browser';
import { of } from 'rxjs';

describe('IframeComponent', () => {
  let component: IframeComponent;
  let fixture: ComponentFixture<IframeComponent>;
  let utilsApiServiceMock: jasmine.SpyObj<UtilsApiService>;
  let routingManagementServiceMock: jasmine.SpyObj<RoutingManagementService>;
  let domSanitizerMock: jasmine.SpyObj<DomSanitizer>;

  beforeEach(async () => {
    // Mock de dependências
    utilsApiServiceMock = jasmine.createSpyObj('UtilsApiService', ['getSelectorsToIgnore', 'getSelectorsForIframeHeightResizing']);
    routingManagementServiceMock = jasmine.createSpyObj('RoutingManagementService', ['getLastPath']);
    domSanitizerMock = jasmine.createSpyObj('DomSanitizer', ['sanitize', 'bypassSecurityTrustHtml']);

    await TestBed.configureTestingModule({
      declarations: [IframeComponent],
      providers: [
        { provide: UtilsApiService, useValue: utilsApiServiceMock },
        { provide: RoutingManagementService, useValue: routingManagementServiceMock },
        { provide: DomSanitizer, useValue: domSanitizerMock }
      ]
    }).compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(IframeComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('deve emitir eventos ao mudar o ssoToken ou url', () => {
    // Mocking the getShadowRootIframe method
    spyOn(component, 'getShadowRootIframe').and.returnValue({
      contentWindow: {
        location: { href: 'http://new-url.com' }
      },
      contentDocument: {
        querySelector: () => ({
          contentWindow: {
            location: { href: 'http://new-url.com' }
          }
        })
      }
    });

    spyOn(component, 'updateIframeHeight');
    spyOn(component, 'eventsAvoidSessionExpired');

    // Simulando mudanças
    const changes: SimpleChanges = {
      ssoToken: {
        previousValue: 'old-token',
        currentValue: 'new-token',
        firstChange: false,
        isFirstChange: () => false
      },
      url: {
        previousValue: 'old-url',
        currentValue: 'new-url',
        firstChange: false,
        isFirstChange: () => false
      }
    };

    // Chamada do método
    component.ngOnChanges(changes);

    // Verificando que os métodos relevantes foram chamados
    expect(component.updateIframeHeight).toHaveBeenCalled();
    expect(component.eventsAvoidSessionExpired).toHaveBeenCalled();
  });

  it('não deve chamar updateIframeHeight se o url e ssoToken não mudarem', () => {
    spyOn(component, 'updateIframeHeight');
    spyOn(component, 'eventsAvoidSessionExpired');

    // Simulando mudanças, mas sem alteração no valor
    const changes: SimpleChanges = {
      ssoToken: {
        previousValue: 'old-token',
        currentValue: 'old-token',  // Não mudou
        firstChange: false,
        isFirstChange: () => false
      },
      url: {
        previousValue: 'old-url',
        currentValue: 'old-url',  // Não mudou
        firstChange: false,
        isFirstChange: () => false
      }
    };

    // Chamada do método
    component.ngOnChanges(changes);

    // Verificando que o método updateIframeHeight não foi chamado
    expect(component.updateIframeHeight).not.toHaveBeenCalled();
    expect(component.eventsAvoidSessionExpired).not.toHaveBeenCalled();
  });
});
