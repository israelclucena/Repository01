describe('IframeComponent - ngOnChanges', () => {
  let component: IframeComponent;
  let fixture: ComponentFixture<IframeComponent>;

  // Mock dos serviços necessários
  const mockRoutingService = jasmine.createSpyObj('RoutingManagementService', ['getLastPath']);
  const mockSanitizer = jasmine.createSpyObj('DomSanitizer', ['sanitize', 'bypassSecurityTrustHtml']);
  const mockUtilsApiService = jasmine.createSpyObj('UtilsApiService', ['getSelectorsToIgnore']);

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [IframeComponent],
      providers: [
        { provide: RoutingManagementService, useValue: mockRoutingService },
        { provide: DomSanitizer, useValue: mockSanitizer },
        { provide: UtilsApiService, useValue: mockUtilsApiService }
      ]
    }).compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(IframeComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('deve chamar updateIframeHeight e eventsAvoidSessionExpired quando ngOnChanges for acionado', () => {
    // Spy nas funções que devem ser chamadas
    spyOn(component, 'updateIframeHeight');
    spyOn(component, 'eventsAvoidSessionExpired');

    // Criar um mock de SimpleChanges para simular mudanças
    const changes: SimpleChanges = {
      url: { previousValue: 'http://old-url.com', currentValue: 'http://new-url.com', firstChange: false, isFirstChange: () => false }
    };

    // Chamar o ngOnChanges manualmente
    component.ngOnChanges(changes);

    // Verificar se os métodos foram chamados corretamente
    expect(component.updateIframeHeight).toHaveBeenCalled();
    expect(component.eventsAvoidSessionExpired).toHaveBeenCalled();
  });
});
