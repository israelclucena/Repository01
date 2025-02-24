import { ComponentFixture, TestBed, fakeAsync, tick } from '@angular/core/testing';
import { IframeComponent } from './iframe.component';
import { DomSanitizer } from '@angular/platform-browser';
import { RoutingManagementService } from 'src/app/services/routing/routing-management.service';
import { UtilsApiService } from 'src/app/services/utils/utils.api.service';
import { CUSTOM_ELEMENTS_SCHEMA, NO_ERRORS_SCHEMA } from '@angular/core';
import { of } from 'rxjs';

describe('IframeComponent', () => {
  let component: IframeComponent;
  let fixture: ComponentFixture<IframeComponent>;
  let mockRouterService: jasmine.SpyObj<RoutingManagementService>;
  let mockUtilsApiService: jasmine.SpyObj<UtilsApiService>;
  let mockSanitizer: jasmine.SpyObj<DomSanitizer>;

  beforeEach(async () => {
    mockRouterService = jasmine.createSpyObj('RoutingManagementService', ['getLastPath']);
    mockUtilsApiService = jasmine.createSpyObj('UtilsApiService', [
      'getSelectorsToIgnore',
      'getSelectorsForIframeHeightResizing',
      'getE2EIFrameHeightResizing',
      'getIFrameUrlDefault',
      'getIFrameHeightSelectorDefault',
      'getIFrameHeightIncreaseDefault'
    ]);
    mockSanitizer = jasmine.createSpyObj('DomSanitizer', ['bypassSecurityTrustHtml', 'sanitize']);

    // Mock de retorno dos serviços
    mockRouterService.getLastPath.and.returnValue(of('/test'));
    mockUtilsApiService.getSelectorsToIgnore.and.returnValue([]);
    mockUtilsApiService.getSelectorsForIframeHeightResizing.and.returnValue([]);
    mockUtilsApiService.getE2EIFrameHeightResizing.and.returnValue([]);

    await TestBed.configureTestingModule({
      declarations: [IframeComponent],
      providers: [
        { provide: RoutingManagementService, useValue: mockRouterService },
        { provide: UtilsApiService, useValue: mockUtilsApiService },
        { provide: DomSanitizer, useValue: mockSanitizer }
      ],
      schemas: [CUSTOM_ELEMENTS_SCHEMA, NO_ERRORS_SCHEMA]
    }).compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(IframeComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create the component', () => {
    expect(component).toBeTruthy();
  });
});
