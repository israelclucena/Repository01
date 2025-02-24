import { ComponentFixture, TestBed } from '@angular/core/testing';
import { IframeComponent } from './iframe.component';
import { SimpleChanges } from '@angular/core';
import { UtilsApiService } from 'src/app/services/utils/utils.api.service';
import { RoutingManagementService } from 'src/app/services/routing/routing-management.service';
import { DomSanitizer } from '@angular/platform-browser';

describe('IframeComponent', () => {
  let component: IframeComponent;
  let fixture: ComponentFixture<IframeComponent>;

  // Serviços Mockados
  let utilsApiServiceMock: jasmine.SpyObj<UtilsApiService>;
  let routingManagementMock: jasmine.SpyObj<RoutingManagementService>;
  let domSanitizerMock: jasmine.SpyObj<DomSanitizer>;

  beforeEach(async () => {
    utilsApiServiceMock = jasmine.createSpyObj('UtilsApiService', ['getSelectorsForIframeHeightResizing']);
    routingManagementMock = jasmine.createSpyObj('RoutingManagementService', ['getLastPath']);
    domSanitizerMock = jasmine.createSpyObj('DomSanitizer', ['sanitize', 'bypassSecurityTrustHtml']);

    await TestBed.configureTestingModule({
      declarations: [IframeComponent],
      providers: [
        { provide: UtilsApiService, useValue: utilsApiServiceMock },
        { provide: RoutingManagementService, useValue: routingManagementMock },
        { provide: DomSanitizer, useValue: domSanitizerMock },
      ]
    }).compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(IframeComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('deve chamar updateIframeHeight e eventsAvoidSessionExpired ao detectar mudanças', () => {
    spyOn(component, 'updateIframeHeight');
    spyOn(component, 'eventsAvoidSessionExpired');

    const changes: SimpleChanges = {
      url: {
        previousValue: 'http://old-url.com',
        currentValue: 'http://new-url.com',
        firstChange: false,
        isFirstChange: () => false
      }
    };

    component.ngOnChanges(changes);

    expect(component.updateIframeHeight).toHaveBeenCalled();
    expect(component.eventsAvoidSessionExpired).toHaveBeenCalled();
  });

  it('não deve chamar updateIframeHeight se a URL não mudar', () => {
    spyOn(component, 'updateIframeHeight');

    const changes: SimpleChanges = {
      ssoToken: {
        previousValue: 'old-token',
        currentValue: 'new-token',
        firstChange: false,
        isFirstChange: () => false
      }
    };

    component.ngOnChanges(changes);

    expect(component.updateIframeHeight).not.toHaveBeenCalled();
  });

  it('deve atualizar a propriedade height corretamente', () => {
    component.height = 200; // Estado anterior

    spyOn(component, 'updateIframeHeight').and.callFake(() => {
      component.height = 300;
    });

    const changes: SimpleChanges = {
      url: {
        previousValue: 'http://old-url.com',
        currentValue: 'http://new-url.com',
        firstChange: false,
        isFirstChange: () => false
      }
    };

    component.ngOnChanges(changes);

    expect(component.height).toBe(300);
  });
});
